```cpp
#pragma once

  

#include <cstdint>

#include <memory>

#include <vector>

#include <string>

#include <map>

#include <unordered_map>

#include <variant>

  

#include "observations/ObservationExtractionGrammar.h"

#include "mapUtilities/MixedTypeMap.h"

  

namespace torch::machinelearning::reinforcementlearning::observation

{

  

using mapUtilities::MixedTypeMap;

  

class IDataExtractionStrategy

{

public:

  

IDataExtractionStrategy(const GrammarInstructionCxt& context) : context_(context) {}

virtual ~IDataExtractionStrategy() = default;

  

virtual MixedTypeMap::RecursiveVariant extract() = 0;

  

protected:

GrammarInstructionCxt context_;

};

  

} // namespace torch::serviceallocator
```