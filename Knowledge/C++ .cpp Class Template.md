```cpp
#include <cstring>

  

#include "yaml-cpp/YamlParser.h"

#include "logger/Logger.h"

#include "logger/LoggerFactory.h"

#include "serviceallocator/ServiceAllocator.h"

#include "serviceallocator/IAllocatorStrategyFactory.h"

#include "serviceallocator/IConfigGeneratorStrategyFactory.h"

  

using toolbox::logger::LoggerFactory;

using toolbox::logger::Logger;

  

namespace torch::serviceallocator

{

  

ServiceAllocator::ServiceAllocator(ConfigInfo config_info)

{

config_ = config_info;

logger_ = LoggerFactory::getLogger();

}

  

void ServiceAllocator::initialize(const int argc, const char* argv[])

{

for (int i = 0; i < argc; ++i)

{

if ((strcmp(argv[i], CMDLINE_LOGGING.c_str()) == 0) && (i + 1 < argc))

{

log_level_ = argv[++i];

}

}

  

logger_->setLogTag(SERVICE_ALLOCATOR_LOGGING_TAG);

logger_->setLogLevel(logger_->stringToLogLevel(log_level_));

}

  

void ServiceAllocator::allocate(uint16_t environment_id, const std::filesystem::path& services_path)

{

// Generation phase

logger_->info("Generating service configs for environment: " + std::to_string(environment_id));

  

std::unordered_map<uint16_t, uint16_t> port_map;

IConfigGeneratorStrategy::StrategyContext generator_context(environment_id, log_level_, config_.services);

IConfigGeneratorStrategyFactory::getStrategy(config_.mode, generator_context)->generate(services_path);

  

logger_->info("Successfully generated service configs for environment: " + std::to_string(environment_id));

  

// Allocation phase

logger_->info("Allocating services for environment: " + std::to_string(environment_id));

  

IAllocatorStrategy::StrategyContext allocator_context(environment_id, log_level_, config_.services);

IAllocatorStrategyFactory::getStrategy(config_.mode, allocator_context)->allocate(services_path);

  

logger_->info("Successfully allocated services for environment: " + std::to_string(environment_id));

}

  

namespace

{

  

template <typename T>

void safeSet(YAML::Node& node, const std::string& key, std::function<void(T)> setter)

{

try

{

if (node[key])

{

setter(node[key].as<T>());

}

else

{

throw std::runtime_error("Key not found in config: " + key);

}

}

catch (const YAML::TypedBadConversion<T>& e)

{

throw std::runtime_error("Invalid type for key: " + key);

}

}

  

}

  

ServiceAllocator::Builder& ServiceAllocator::Builder::setWithYaml(const std::string &file_path)

{

#ifdef USE_YAML_CPP

Torch::ADVANCE::YamlParser parser = Torch::ADVANCE::YamlParser();

YAML::Node node = parser.parse(file_path);

  

safeSet<std::string>(node, CONFIG_PARAM_MODE, [this](std::string mode) { this->setMode(mode); });

  

switch (config.mode)

{

case Mode::LOCAL:

{

YAML::Node local_node = node[CONFIG_PARAM_LOCAL];

safeSet<std::map<std::string, std::map<std::string, std::string>>>(local_node, CONFIG_PARAM_SERVICES, [this](std::map<std::string, std::map<std::string, std::string>> services) { this->setServices(services); });

break;

}

}

#endif

  

return *this;

}

  

ServiceAllocator::Builder& ServiceAllocator::Builder::setMode(std::string mode)

{

try

{

config.mode = mode_map.at(mode);

}

catch (const std::out_of_range& e)

{

throw std::runtime_error("Invalid mode: " + mode);

}

return *this;

}

  

ServiceAllocator::Builder& ServiceAllocator::Builder::setServices(std::map<std::string, std::map<std::string, std::string>> services)

{

for (const std::pair<std::string, std::map<std::string, std::string>>& service_map : services)

{

Service service;

  

if (service_map.second.find(CONFIG_PARAM_EXECUTION) != service_map.second.end())

{

try

{

service.execution = execution_map.at(service_map.second.at(CONFIG_PARAM_EXECUTION));

}

catch (const std::out_of_range& e)

{

throw std::runtime_error("Invalid value for field " + CONFIG_PARAM_EXECUTION + " for service: " + service_map.first + ". Valid values are " + CONFIG_VALUE_AUTOMATIC + " and " + CONFIG_VALUE_MANUAL);

}

  

if (service.execution == Execution::AUTOMATIC)

{

if (service_map.second.find(CONFIG_PARAM_EXECUTABLE) != service_map.second.end())

{

service.executable = service_map.second.at(CONFIG_PARAM_EXECUTABLE);

}

else

{

throw std::runtime_error("Missing required field " + CONFIG_PARAM_EXECUTABLE + " for service: " + service_map.first);

}

}

}

else

{

throw std::runtime_error("Missing required field " + CONFIG_PARAM_EXECUTION + " for service: " + service_map.first);

}

config.services[service_map.first] = service;

}

  

return *this;

}

  

ServiceAllocator ServiceAllocator::Builder::build()

{

return ServiceAllocator(config);

}

  

} // namespace torch::serviceallocator
```