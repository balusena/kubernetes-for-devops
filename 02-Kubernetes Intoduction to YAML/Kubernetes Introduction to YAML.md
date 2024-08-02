# Introduction to YAML (YAML Ain't Markup Language)
YAML is a human-readable data serialization standard commonly used for configuration files and data 
exchange between languages with different data structures. It is known for its simplicity and readability, 
emphasizing data over document markup.

## Key Features

### 1.Human-Readable
YAML syntax is designed to be easy to read and write, with a minimalistic style that avoids the complexity of XML or JSON.

### 2.Data Structures
YAML supports complex data structures like lists, dictionaries (mappings), and scalars (strings, numbers, booleans).

### 3.Indentation-Based
YAML uses indentation to denote structure, making it visually clear. Consistent indentation is crucial.

### 4.Support for Comments
Comments in YAML start with a `#`, allowing for documentation within configuration files.

### 5.Flexible and Powerful
It can represent a wide range of data types and supports references and anchors to avoid redundancy.

**YAML is a Serialization Language**
```
#Serialization:

              Serialization                                            Deserialization
Source ----------------------------> [0011000100011100000001] -------------------------------> Destination 

           {                                  Network
             "name": "John",
             "age": 30,
             "city": "New York"
           }

* The process of converting an object to binary is called serialization.

* The process of converting binary back to the original data is called deserialization.

```
```
#Deserialization:

                  Deserialization                                          Serialization
Binary Data ---------------------------->    {                        ---------------------> Original Data
              [0011000100011100000001]            "name": "John",
                                                  "age": 30,
                                                  "city": "New York"
                                             } 

                                                       Network

* The process of converting binary back to the original data is called deserialization.

* The process of converting an object to binary is called serialization.
```