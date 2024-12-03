```cpp
#pragma once

  

#include <string>

#include <cstdint>

#include <stdexcept>

#include <map>

#include <filesystem>

#include <vector>

#include <unordered_map>

#include <variant>

  

#include "serviceallocator/ServiceAllocatorConstants.h"

  

namespace toolbox::logger {

class Logger;

}

  

namespace torch::serviceallocator

{

  

enum class Execution

{

UNKNOWN,

MANUAL,

AUTOMATIC

};

  

enum class ExecutionLocation

{

UNKNOWN,

LOCAL,

};

  

struct Service

{

std::string executable;

Execution execution = Execution::UNKNOWN;

};

  

class ServiceAllocator

{

public:

  

enum class Mode

{

UNKNOWN,

LOCAL,

};

  

struct ConfigInfo

{

Mode mode = Mode::UNKNOWN;

std::map<std::string, Service> services = {};

};

  
  

ServiceAllocator(ConfigInfo config_info);

~ServiceAllocator() = default;

  

ServiceAllocator(const ServiceAllocator& other) = default; // Copy constructor

ServiceAllocator(ServiceAllocator&& other) = default; // Move constructor

ServiceAllocator& operator=(const ServiceAllocator& other) = default; // Copy Assignment operator

ServiceAllocator& operator=(ServiceAllocator&& other) = default; // Move Assignment operator

  

class Builder

{

public:

Builder& setWithYaml(const std::string& file_path);

Builder& setMode(std::string mode);

Builder& setServices(std::map<std::string, std::map<std::string, std::string>> services);

ServiceAllocator build();

  

private:

const std::unordered_map<std::string, Mode> mode_map = {{CONFIG_PARAM_LOCAL, Mode::LOCAL}};

const std::unordered_map<std::string, Execution> execution_map = {{CONFIG_VALUE_MANUAL, Execution::MANUAL},

{CONFIG_VALUE_AUTOMATIC, Execution::AUTOMATIC}};

  

ConfigInfo config;

};

  

// Getters

inline Mode getMode() const { return config_.mode; }

  

/**

* @brief initialize the service allocator with the command line arguments

*

* @param argc

* @param argv

*/

void initialize(const int argc, const char* argv[]);

  

/**

* @brief Generates service configs and then allocates the services for the current environment

*

* @param environment_id

* @param services_path

*/

void allocate(uint16_t environment_id, const std::filesystem::path& services_path);

  

private:

  

ConfigInfo config_;

std::shared_ptr<toolbox::logger::Logger> logger_;

std::string log_level_ = "info";

};

  

} // namespace torch::serviceallocator
```