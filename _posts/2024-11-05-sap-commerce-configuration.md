---
title: "SAP Commerce (Hybris): Check Current Configuration"
last_modified_at: 2024-11-07T16:20:02-05:00
categories:
  - SAP Commerce
tags:
  - SAP Commerce
  - Hybris
  - Groovy
classes: wide
---

# Check SAP Commerce Configuration

Sometimes a need could arise to change some properties in HAC to test some behavior on development machines. Then you wanna know if the change was picked up by SAP Commerce. It can also happen that you ask yourself if the some properties did deviate from the configured values. For such kind of use cases it would be  very beneficial to have a small script to get a quick answer.

Below is a Groovy script, which reports if a configuration did drift away by comparing it against the runtime value. 

```groovy

import de.hybris.platform.servicelayer.config.ConfigurationService
import de.hybris.platform.util.Config
import java.nio.file.Files
import java.nio.file.Paths

def loadFromPropertiesFile(String propertiesPath, String propertyKey) {
    def propertiesFile = new File(propertiesPath)

    if (propertiesFile.exists()) {
        def props = new Properties()
        propertiesFile.withInputStream { props.load(it) }

        return props.getProperty(propertyKey)
    }

    throw new FileNotFoundException("Properties file doesn't exist")
}

def propertyKeyFromLocalProperties(String propertyKey) {
    def filePath = Config.getParameter("HYBRIS_CONFIG_DIR") + "/local.properties"

    return loadFromPropertiesFile(filePath, propertyKey)
}

def propertyKeyFromProjectProperties(String propertyKey) {
    def filePath = Config.getParameter("HYBRIS_BIN_DIR") + "/platform/project.properties"

    return loadFromPropertiesFile(filePath, propertyKey)
}

def propertyKeyRuntimeValue(String propertyKey) {
    return Config.getParameter(propertyKey)
}

def propertyKeyFromConfigurationService(String propertyKey) {
    configurationService.getConfiguration().getString(propertyKey)
}

def propertyKeyFromSystemProperties(String propertyKey) {
    return System.getProperty(propertyKey)
}

def analyzeConfigurationPropertyState(String propertyKey) {
    def localPropertyValue = propertyKeyFromLocalProperties(propertyKey)
    def projectPropertyValue = propertyKeyFromProjectProperties(propertyKey)
    def runtimeValue = propertyKeyRuntimeValue(propertyKey)
    def configServiceValue = propertyKeyFromConfigurationService(propertyKey)
    def systemPropertyValue = propertyKeyFromSystemProperties(propertyKey)

    println "Configuration Status for: ${propertyKey}"
    println "----------------------------------------"
    println "1. local.properties value:     ${localPropertyValue}"
    println "2. project.properties value:   ${projectPropertyValue}"
    println "3. Current runtime value:      ${runtimeValue}"
    println "4. ConfigurationService value: ${configServiceValue}"
    println "5. System property value:      ${systemPropertyValue}"
    println "----------------------------------------"

    // Analyze if restart might be needed
    def needsRestart = false
    def explanation = []

    // Needs restart if local.properties differs from runtime
    if (localPropertyValue != null && localPropertyValue != runtimeValue) {
        needsRestart = true
        explanation << "local.properties value differs from runtime value"
    }

    // Needs restart if project.properties differs from runtime
    if (projectPropertyValue != null && localPropertyValue == null && projectPropertyValue != runtimeValue) {
        needsRestart = true
        explanation << "project.properties value differs from runtime value"
    }

    println "\nAnalysis:"
    if (needsRestart) {
        println "⚠️ RESTART NEEDED because: ${explanation.join(', ')}"
    } else {
        println "✅ No restart needed - configuration is active"
    }

    // Additional checks for common scenarios
    if (runtimeValue != configServiceValue) {
        println "⚠️ Warning: Runtime value differs from ConfigurationService value - possible configuration override"
    }

    if (systemPropertyValue != null) {
        println "ℹ️ Note: Property is set as system property, which takes precedence over properties files"
    }
}

/////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////// USAGE ////////////////////////////////////////////////
// Example usage:
def propertyToCheck = "db.url"
analyzeConfigurationPropertyState(propertyToCheck)

```

### Usage

Set the property you're interested in before executing the script in HAC.

### A Quick Explanation

When starting up the SAP Commerce Platform the system tries to load the system resource `project.properties` which is normally inside the `platform` directory or the `local.properties` which is normally at the `config` folder. This file contains entries of key value pairs separated by `=`, e.g. `key=value`. Settings in `local.properties` override `project.properties`.

For checking the runtime values, we utilize `de.hybris.platform.util.Config`, which is the main class that is used to load and store SAP Commerce platform configuration settings.



