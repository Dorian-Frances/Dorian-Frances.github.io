---
layout: post
title: "The Power of .ini Files with configparser in Python: A Superior Alternative to .env Files"
date: 2023-06-14
tags: [configuration, python]
reading-time: 10
description: Configuration files are crucial in software development as they store essential parameters and settings that allow applications to function effectively in different environments. These files separate configuration variables from the code, offering flexibility, security, and ease of maintenance.
comments: true
---

# Introduction

Configuration files are crucial in software development as they store essential parameters and settings that allow applications to function effectively in different environments. These files separate configuration variables from the code, offering flexibility, security, and ease of maintenance.

## Importance of Configuration Files

1. *Separation of Concerns*: Configuration files isolate application logic from environment-specific details, promoting modularity and simplifying deployment and maintenance.
2. *Flexibility*: Configuration files enable easy adjustment of application behavior without code changes, facilitating adaptation to various environments.
3. *Security*: Sensitive information, like credentials, remains secure by storing it in configuration files instead of hard-coding it within the codebase.
4. *Collaboration*: Centralizing configuration variables in files enhances teamwork and allows efficient adjustments based on specific requirements.
5. *Maintainability*: Configuration files simplify updates and troubleshooting, making it easier to manage and modify variables when needed.

## Limitations of `.env` files

While .env files have gained popularity for storing configuration variables, they do have limitations and challenges that developers should be aware of:

1. *Limited Data Types*: .env files primarily support string values, which can be limiting when working with configuration variables that require different data types, such as integers, booleans, or complex structures. Converting and parsing values from strings to the desired data type can be cumbersome and error-prone.
2. *Lack of Structure*: .env files lack a built-in structure for organizing configuration variables. As the number of variables grows, maintaining organization and readability becomes increasingly challenging. Developers often resort to naming conventions or comments to provide some level of organization, but this approach is not standardized or enforced.
3. *Absence of Hierarchy*: .env files lack a hierarchical structure, making it difficult to group related variables together or define subsections. This can result in a flat and cluttered configuration file, especially when managing multiple environments or complex settings.
4. *Limited Validation and Error Handling*: .env files do not provide inherent validation or error handling mechanisms. It is the responsibility of the developer to ensure the correctness and integrity of the variables. Without proper validation, it's possible to introduce errors or inconsistencies that can cause application failures or unexpected behavior.

Considering these limitations and challenges, developers often seek alternatives like .ini files with *configparser* in Python, which offer a more robust and structured approach to configuration management. In the following sections, we'll explore the advantages of using .ini files and *configparser*, addressing these limitations and providing a superior solution for configuration handling in Python.

# Introducing `.ini` files and configparser

.ini files are configuration files with a specific structure and syntax. They consist of sections, keys, and values. Here's a simple example:

```toml
# config.ini

[section1]
key1 = value1
key2 = value2

[section2]
key3 = value3
key4 = value4
```

In Python, the **`configparser`** module can be used to read values from .ini files. Here's a minimal code example:

```python
import configparser

config = configparser.ConfigParser()
config.read('config.ini')

value1 = config.get('section1', 'key1')
value2 = config.get('section1', 'key2')
value3 = config.get('section2', 'key3')
value4 = config.get('section2', 'key4')
```

In this brief example, we import the **`configparser`** module, create a **`ConfigParser`** instance, and read the content of the **`config.ini`** file. We then retrieve values from specific sections and keys using the **`get()`** method.

# Advantages of `.ini` files with configparser

## Simplicity and Readability

```python
import configparser

# Read values from .ini file
config = configparser.ConfigParser()
config.read('config.ini')

value = config.get('section', 'key')

# Modify values and write back to .ini file
config.set('section', 'key', 'new_value')
config.write(open('config.ini', 'w'))
```

In this example, we import the **`configparser`** module, create a **`ConfigParser`** instance, and read values from the **`config.ini`** file using the **`get()`** method.

To modify values, we use the **`set()`** method to update a specific key within a section. Then, we write the changes back to the .ini file using the **`write()`** method.

*configparser* handles the complexities of parsing, accessing, modifying, and writing .ini files, simplifying the process for us (working with configuration values is hard enough as it is).

## Hierarchical Structure

.ini files can be organized into sections and subsections, providing a hierarchical structure for configuration variables.

```toml
[section1]
key1 = value1
key2 = value2

[section2]
key3 = value3

[sub_section]
key4 = value4
```

In this example, we have two sections, **`[section1]`** and **`[section2]`**. Additionally, there is a subsection **`[sub_section]`** nested within **`[section2]`**.

By organizing configuration variables into sections and subsections, we can easily group related variables together, improving clarity and organization.

To access `value4` , we then just need to write a few simple lines of code: 

```python
import configparser

config = configparser.ConfigParser()
config.read('config.ini')

value4 = config.get('section2.sub_section', 'key4')
```

## Multiple Value Types

*configparser* in Python supports various data types for configuration values, such as strings, integers, booleans, and floats. 

```python
value_int = config.getint('section', 'key')
value_bool = config.getboolean('section', 'key')
value_float = config.getfloat('section', 'key')
```

By using the appropriate **`get`** methods provided by configparser (**`getint()`**, **`getboolean()`**, **`getfloat()`**), we can easily retrieve configuration values in the desired data type without the need for manual conversion. This flexibility in handling different data types simplifies the process of working with configuration variables of various types in Python applications.

## Error Handling and Validation

*configparser* in Python provides built-in error handling and validation mechanisms to ensure the correctness and integrity of configuration data. The main useful features could be sum up as follows:

1. *Missing Section or Key Handling*: When accessing configuration values, configparser raises an error if a specified section or key is not found in the .ini file. This helps to catch and handle situations where expected configuration variables are missing.
2. *Syntax Error Detection*: configparser detects and reports syntax errors in .ini files. If the file has incorrect syntax or formatting, configparser raises an error indicating the location and nature of the syntax error. This aids in identifying and resolving syntax-related issues in configuration files.
3. *Type Conversion Errors*: As we said earlier in this article, *configparser* support typing verification. Thus, when using the specialized **`get`** methods (e.g., **`getint()`**, **`getboolean()`**, **`getfloat()`**), configparser automatically performs type conversions. If a configuration value cannot be converted to the requested type, configparser raises a type conversion error, preventing incorrect data usage.
4. *Duplicate Sections and Keys*: *configparser* ensures that sections and keys in .ini files are unique. If a duplicate section or key is encountered, configparser raises an error, preventing ambiguity or conflicts in configuration data.

# Practical example for the use of `.ini` files

For example, you could have at the root of your project two directories `properties` and `secrets`  as follows:

```
properties
├── local.ini
└── properties.ini
```

```
secrets
├── local.ini
└── secrets.ini
```

We just make a distinction for *properties* and *secrets* here but both folders refer configuration values. 

In your **`properties/local.ini`** (values used for local work) you will have:

```toml
[POSTGRES]
postgres_username = postgres
postgres_host = localhost
postgres_port = 5432
postgres_db = database
```

In your **`properties/properties.ini`** (values used for production), you will have:

```toml
[POSTGRES]
postgres_username = %(POSTGRES_USER)s
postgres_host = %(POSTGRES_HOST)s
postgres_port = %(POSTGRES_PORT)s
postgres_db = %(POSTGRES_DB)s
```

> 👉 This notation is called **interporlation** It tells *configparser* that “POSTGRES_USER” refers to either, another variable present in the `.ini` file or an environment variable.
> 
> In the second case, to access that value in the code, you would need to call the variable as follows:
> 
> `config.get('POSTGRES', 'postgres_username', **vars=os.environ**)`


When you want to deploy your application on your server, you then, can set your **environment variables values** in you CI/CD (e.g. CircleCI) and create a script to export them on your server when deploying so that your code can access it.  

# Conclusion

`.env` files are convenient, for sure, as they are generally well implemented in many available frameworks. But `.ini` files offer a lot of powerful features, such as hierarchical structure, data validation, error handling, multiple value types, etc… 

Those make the use of `.ini` files really intuitive and robust, that is why, I, as a software engineer advise people to use them. 

The difference of use may appear insignificant, but those configuration values are, more often thant you think, the source of production regression. You may forget to set a value ? Or more subtil, you may set `"5432"` instead of `5432` as your database port value ?

Being an effective developer is about reducing, as much as possible, our feedback loop. When something is not supposed to work, we appreciate knowing it as soon as possible and `.ini` features such as *data validation* (among others) could make a difference in your development efficiency.