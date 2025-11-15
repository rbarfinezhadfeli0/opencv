# Documentation for `modules/core/src/utils/logtagconfigparser.cpp`

## File Metadata

- **Full Path**: `modules/core/src/utils/logtagconfigparser.cpp`
- **File Name**: `logtagconfigparser.cpp`
- **File Size**: 8,480 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/src/utils/logtagconfigparser.cpp](../../../../modules/core/src/utils/logtagconfigparser.cpp)

## Purpose and Role

This file is located in the `modules/core/src/utils` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "../precomp.hpp"
#include "logtagconfigparser.hpp"

namespace cv {
namespace utils {
namespace logging {

LogTagConfigParser::LogTagConfigParser(LogLevel defaultUnconfiguredGlobalLevel)
{
    m_parsedGlobal.namePart = "global";
    m_parsedGlobal.isGlobal = true;
    m_parsedGlobal.hasPrefixWildcard = false;
    m_parsedGlobal.hasSuffixWildcard = false;
    m_parsedGlobal.level = defaultUnconfiguredGlobalLevel;
}

LogTagConfigParser::LogTagConfigParser(const std::string& input)
{
    parse(input);
}

LogTagConfigParser::~LogTagConfigParser()
{
}

bool LogTagConfigParser::parse(const std::string& input)
{
    m_input = input;
    segmentTokens();
    return (m_malformed.empty());
}

bool LogTagConfigParser::hasMalformed() const
{
    return !m_malformed.empty();
}

const LogTagConfig& LogTagConfigParser::getGlobalConfig() const
{
    return m_parsedGlobal;
}

const std::vector<LogTagConfig>& LogTagConfigParser::getFullNameConfigs() const
{
    return m_parsedFullName;
}

const std::vector<LogTagConfig>& LogTagConfigParser::getFirstPartConfigs() const
{
    return m_parsedFirstPart;
}

const std::vector<LogTagConfig>& LogTagConfigParser::getAnyPartConfigs() const
{
    return m_parsedAnyPart;
}

const std::vector<std::string>& LogTagConfigParser::getMalformed() const
{
    return m_malformed;
}

void LogTagConfigParser::segmentTokens()
{
    const size_t len = m_input.length();
    std::vector<std::pair<size_t, size_t>> startStops;
    bool wasSeparator = true;
    for (size_t pos = 0u; pos < len; ++pos)
    {
        char c = m_input[pos];
        bool isSeparator = (c == ' ' || c == '\t' || c == ';');
        if (!isSeparator)
        {
            if (wasSeparator)
            {
                startStops.emplace_back(pos, pos + 1u);
            }
            else
            {
                startStops.back().second = pos + 1u;
            }
        }
        wasSeparator = isSeparator;
    }
    for (const auto& startStop : startStops)
    {
        const auto s = m_input.substr(startStop.first, startStop.second - startStop.first);
        parseNameAndLevel(s);
    }
}

void LogTagConfigParser::parseNameAndLevel(const std::string& s)
{
    const size_t npos = std::string::npos;
    const size_t len = s.length();
    size_t colonIdx = s.find_first_of(":=");
    if (colonIdx == npos)
    {
        // See if the whole string is a log level
        auto parsedLevel = parseLogLevel(s);
        if (parsedLevel.second)
        {
            // If it is, assume the log level is for global
            parseWildcard("", parsedLevel.first);
            return;
        }
        else
        {
            // not sure what to do.
            m_malformed.push_back(s);
            return;
        }
    }
    if (colonIdx == 0u || colonIdx + 1u == len)
    {
        // malformed (colon or equal sign at beginning or end), cannot do anything
        m_malformed.push_back(s);
        return;
    }
    size_t colonIdx2 = s.find_first_of(":=", colonIdx + 1u);
    if (colonIdx2 != npos)
    {
        // malformed (more than one colon or equal sign), cannot do anything
        m_malformed.push_back(s);
        return;
    }
    auto parsedLevel = parseLogLevel(s.substr(colonIdx + 1u));
    if (parsedLevel.second)
    {
        parseWildcard(s.substr(0u, colonIdx), parsedLevel.first);
        return;
    }
    else
    {
        // Cannot recognize the right side of the colon or equal sign.
        // Not sure what to do.
        m_malformed.push_back(s);
        return;
    }
}

void LogTagConfigParser::parseWildcard(const std::string& name, LogLevel level)
{
    const size_t npos = std::string::npos;
    const size_t len = name.length();
    if (len == 0u)
    {
        m_parsedGlobal.level = level;
        return;
    }
    const bool hasPrefixWildcard = (name[0u] == '*');
    if (hasPrefixWildcard && len == 1u)
    {
        m_parsedGlobal.level = level;
        return;
    }
    const size_t firstNonWildcard = name.find_first_not_of("*.");
    if (hasPrefixWildcard && firstNonWildcard == npos)
    {
        m_parsedGlobal.level = level;
        return;
    }
    const bool hasSuffixWildcard = (name[len - 1u] == '*');
    const size_t lastNonWildcard = name.find_last_not_of("*.");
    std::string trimmedNamePart = name.substr(firstNonWildcard, lastNonWildcard - firstNonWildcard + 1u);
    // The case of a single asterisk has been handled above;
    // here we only handle the explicit use of "global" in the log config string.
    const bool isGlobal = (trimmedNamePart == "global");
    if (isGlobal)
    {
        m_parsedGlobal.level = level;
        return;
    }
    LogTagConfig result(trimmedNamePart, level, false, hasPrefixWildcard, hasSuffixWildcard);
    if (hasPrefixWildcard)
    {
        m_parsedAnyPart.emplace_back(std::move(result));
    }
    else if (hasSuffixWildcard)
    {
        m_parsedFirstPart.emplace_back(std::move(result));
    }
    else
    {
        m_parsedFullName.emplace_back(std::move(result));
    }
}

std::pair<LogLevel, bool> LogTagConfigParser::parseLogLevel(const std::string& s)
{
    const auto falseDontCare = std::make_pair(LOG_LEVEL_VERBOSE, false);
    const auto make_parsed_result = [](LogLevel lev) -> std::pair<LogLevel, bool>
    {
        return std::make_pair(lev, true);
    };
    const size_t len = s.length();
    if (len >= 1u)
    {
        const char c = (char)std::toupper(s[0]);
        switch (c)
        {
        case '0':
            if (len == 1u)
            {
                return make_parsed_result(LOG_LEVEL_SILENT);
            }
            break;
        case 'D':
            if (len == 1u ||
                (len == 5u && cv::toUpperCase(s) == "DEBUG"))
            {
                return make_parsed_result(LOG_LEVEL_DEBUG);
            }
            if ((len == 7u && cv::toUpperCase(s) == "DISABLE") ||
                (len == 8u && cv::toUpperCase(s) == "DISABLED"))
            {
                return make_parsed_result(LOG_LEVEL_SILENT);
            }
            break;
        case 'E':
            if (len == 1u ||
                (len == 5u && cv::toUpperCase(s) == "ERROR"))
            {
                return make_parsed_result(LOG_LEVEL_ERROR);
            }
            break;
        case 'F':
            if (len == 1u ||
                (len == 5u && cv::toUpperCase(s) == "FATAL"))
            {
                return make_parsed_result(LOG_LEVEL_FATAL);
            }
            break;
        case 'I':
            if (len == 1u ||
                (len == 4u && cv::toUpperCase(s) == "INFO"))
            {
                return make_parsed_result(LOG_LEVEL_INFO);
            }
            break;
        case 'O':
            if (len == 3u && cv::toUpperCase(s) == "OFF")
            {
                return make_parsed_result(LOG_LEVEL_SILENT);
            }
            break;
        case 'S':
            if (len == 1u ||
                (len == 6u && cv::toUpperCase(s) == "SILENT"))
            {
                return make_parsed_result(LOG_LEVEL_SILENT);
            }
            break;
        case 'V':
            if (len == 1u ||
                (len == 7u && cv::toUpperCase(s) == "VERBOSE"))
            {
                return make_parsed_result(LOG_LEVEL_VERBOSE);
            }
            break;
        case 'W':
            if (len == 1u ||
                (len == 4u && cv::toUpperCase(s) == "WARN") ||
                (len == 7u && cv::toUpperCase(s) == "WARNING") ||
                (len == 8u && cv::toUpperCase(s) == "WARNINGS"))
            {
                return make_parsed_result(LOG_LEVEL_WARNING);
            }
            break;
        default:
            break;
        }
        // fall through
    }
    return falseDontCare;
}

std::string LogTagConfigParser::toString(LogLevel level)
{
    switch (level)
    {
    case LOG_LEVEL_SILENT:
        return "SILENT";
    case LOG_LEVEL_FATAL:
        return "FATAL";
    case LOG_LEVEL_ERROR:
        return "ERROR";
    case LOG_LEVEL_WARNING:
        return "WARNING";
    case LOG_LEVEL_INFO:
        return "INFO";
    case LOG_LEVEL_DEBUG:
        return "DEBUG";
    case LOG_LEVEL_VERBOSE:
        return "VERBOSE";
    default:
        return std::to_string((int)level);
    }
}

}}} //namespace
```

## High-Level Overview

This is a C++ implementation file containing the core logic and algorithms for OpenCV functionality.

**Key Characteristics:**
- Implements algorithms and data processing routines
- May contain performance-critical code
- Uses C++ features like templates, classes, and STL
- Integrates with OpenCV's module system


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `logtagconfigparser.hpp`
- `../precomp.hpp`


### Architectural Role

This file operates within the OpenCV module system, interfacing with other components through well-defined APIs and data structures.

## Performance and Complexity

### Computational Complexity

The algorithms and data structures in this file have various complexity characteristics depending on the operations performed.

### Memory Considerations

Memory usage patterns depend on the specific functionality implemented, including stack allocations, heap allocations, and resource management strategies.

### Performance Optimization

OpenCV employs various optimization techniques including:
- SIMD vectorization where applicable
- Multi-threading support
- Hardware acceleration (CUDA, OpenCL, etc.)
- Efficient memory access patterns

## Security and Safety Considerations

### Potential Vulnerabilities

Code that processes external data should be carefully reviewed for:
- Buffer overflow vulnerabilities
- Integer overflow/underflow
- Input validation issues
- Resource exhaustion attacks

### Safety Measures

OpenCV includes various safety mechanisms:
- Bounds checking in debug builds
- Exception handling
- Resource management (RAII in C++)
- Input sanitization

## Testing and Usage

### How to Use This File

This file is typically used as part of the larger OpenCV library and is not intended to be used in isolation.

### Testing Approach

Testing should cover:
- Unit tests for individual functions
- Integration tests for component interactions
- Performance benchmarks
- Edge case validation

## Related Files

This file is related to other files in the same module and may interact with files in other modules.

