# 🐍 Python Descriptors Guide

## 📖 Complete Guide to Understanding and Using Descriptors

---

## 📑 Table of Contents

### **Section 1: Introduction & Fundamentals** 🎯 *(Current)*
- [What Are Descriptors?](#what-are-descriptors)
- [Why Use Descriptors?](#why-use-descriptors)
- [The Descriptor Protocol](#the-descriptor-protocol)
- [Simple Example: Constant Descriptor](#simple-example-constant-descriptor)
- [Dynamic Lookups](#dynamic-lookups)
- [When Descriptors Are Invoked](#when-descriptors-are-invoked)
- [Key Concepts Summary](#key-concepts-summary)

### **Section 2: Descriptor Protocol Deep Dive** 🔬 *(Coming)*
### **Section 3: Practical Applications** 💼 *(Coming)*
### **Section 4: Built-in Descriptors Explained** 🏗️ *(Coming)*
### **Section 5: Best Practices & Patterns** ✨ *(Coming)*

---

<a id="what-are-descriptors"></a>
## 🎯 What Are Descriptors?

**Descriptors** are a powerful Python feature that allows objects to customize **attribute lookup**, **storage**, and **deletion**. They provide a way to add managed attributes to classes with custom behavior.

### 🔑 Core Definition

A descriptor is any object that defines **one or more** of these special methods:

- `__get__(self, obj, objtype=None)` - Called to get an attribute
- `__set__(self, obj, value)` - Called to set an attribute  
- `__delete__(self, obj)` - Called to delete an attribute

### 📊 Descriptor Architecture Overview

```mermaid
graph TB
    Start[Attribute Access<br/><code>obj.attr</code>]
    
    subgraph lookup["ATTRIBUTE LOOKUP ORDER"]
        L1{Check type<br/>__dict__}
        L2{Has __get__<br/>and __set__?}
        L3{Check instance<br/>__dict__}
        L4{Check type<br/>__dict__ again}
        L5{Has __get__<br/>only?}
        L6[Walk MRO chain]
    end
    
    subgraph descriptors["DESCRIPTOR TYPES"]
        DD["<b>Data Descriptor</b><br/>Has __set__ or __delete__<br/>Takes priority"]
        ND["<b>Non-Data Descriptor</b><br/>Only has __get__<br/>Lower priority"]
    end
    
    subgraph protocol["DESCRIPTOR PROTOCOL"]
        G["<b>__get__(self, instance, owner)</b><br/>Called on attribute access<br/>instance: obj or None<br/>owner: type(obj)"]
        S["<b>__set__(self, instance, value)</b><br/>Called on assignment<br/>Prevents instance __dict__ storage"]
        D["<b>__delete__(self, instance)</b><br/>Called on deletion<br/>del obj.attr"]
    end
    
    subgraph examples["BUILT-IN EXAMPLES"]
        P["<b>property</b><br/>Data descriptor<br/>@property decorator"]
        CM["<b>classmethod</b><br/>Non-data descriptor<br/>@classmethod decorator"]
        SM["<b>staticmethod</b><br/>Non-data descriptor<br/>@staticmethod decorator"]
        F["<b>Functions</b><br/>Non-data descriptor<br/>Bound method creation"]
    end
    
    subgraph results["BEHAVIOR"]
        R1[Invoke __get__<br/>Return computed value]
        R2[Instance __dict__<br/>lookup succeeds]
        R3[Invoke __get__<br/>from class]
        R4[AttributeError<br/>not found]
    end
    
    Start --> L1
    L1 -->|Found descriptor| L2
    L2 -->|Yes - Data Descriptor| DD
    DD --> G
    DD -.->|if present| S
    DD -.->|if present| D
    
    L2 -->|No| L3
    L3 -->|Found in instance| R2
    L3 -->|Not found| L4
    
    L4 -->|Found descriptor| L5
    L5 -->|Yes - Non-Data| ND
    ND --> G
    
    L5 -->|No| L6
    L6 -->|Found| R3
    L6 -->|Not found| R4
    
    G --> R1
    
    P -.->|implements| DD
    CM -.->|implements| ND
    SM -.->|implements| ND
    F -.->|implements| ND
    
    classDef start fill:#e1f5fe,stroke:#01579b,stroke-width:3px
    classDef lookup fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef dataDesc fill:#c8e6c9,stroke:#2e7d32,stroke-width:3px
    classDef nonDataDesc fill:#b2dfdb,stroke:#00695c,stroke-width:2px
    classDef protocol fill:#f3e5f5,stroke:#6a1b9a,stroke-width:2px
    classDef example fill:#ffe0b2,stroke:#e65100,stroke-width:2px
    classDef result fill:#e1bee7,stroke:#7b1fa2,stroke-width:2px
    
    class Start start
    class L1,L2,L3,L4,L5,L6 lookup
    class DD dataDesc
    class ND nonDataDesc
    class G,S,D protocol
    class P,CM,SM,F example
    class R1,R2,R3,R4 result
```

[↑ Back to TOC](#-table-of-contents)

---

<a id="why-use-descriptors"></a>
## 💡 Why Use Descriptors?

Descriptors provide several powerful capabilities that make Python code more elegant and maintainable:

### ✨ Key Benefits

| Benefit | Description | Example Use Case |
|---------|-------------|------------------|
| 🔒 **Attribute Control** | Intercept attribute access to add custom logic | Validation, logging, computed values |
| 🎭 **Encapsulation** | Hide implementation details behind clean interfaces | Private attributes with public access |
| ♻️ **Reusability** | Create reusable attribute behaviors | Type checking across multiple classes |
| 🧹 **DRY Principle** | Avoid repeating attribute management code | Single validator for many attributes |
| 🏗️ **Framework Building** | Foundation for properties, methods, ORM tools | Django models, SQLAlchemy |

### 🌟 Real-World Applications

Descriptors are used throughout Python itself for:

- ✅ **Properties** - The `@property` decorator
- ✅ **Methods** - How functions become bound methods
- ✅ **Static/Class Methods** - `@staticmethod` and `@classmethod`
- ✅ **Cached Properties** - `@functools.cached_property`
- ✅ **ORM Systems** - Database field definitions
- ✅ **Type Validation** - Runtime type checking

[↑ Back to TOC](#-table-of-contents)

---

<a id="the-descriptor-protocol"></a>
## 🔌 The Descriptor Protocol

The descriptor protocol consists of three optional methods that control attribute access:

### 📋 Protocol Methods

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        """Called when the descriptor is accessed
        
        Args:
            self: The descriptor instance
            obj: The instance accessing the descriptor (or None for class access)
            objtype: The type of the instance (the class)
        
        Returns:
            The value to return for the attribute access
        """
        pass
    
    def __set__(self, obj, value):
        """Called when the descriptor is set
        
        Args:
            self: The descriptor instance
            obj: The instance setting the descriptor
            value: The value being assigned
        """
        pass
    
    def __delete__(self, obj):
        """Called when the descriptor is deleted
        
        Args:
            self: The descriptor instance
            obj: The instance deleting the descriptor
        """
        pass
```

### 🎨 Method Combinations

```mermaid
graph LR
    subgraph TYPES ["📦    Descriptor    Types"]
        A1[Only __get__]
        A2[__get__ + __set__]
        A3[__get__ + __delete__]
        A4[All Three Methods]
    end
    
    subgraph CLASSIFICATION ["🏷️    Classification"]
        B1[Non-Data<br/>Descriptor]
        B2[Data<br/>Descriptor]
        B3[Data<br/>Descriptor]
        B4[Full<br/>Descriptor]
    end
    
    subgraph BEHAVIOR ["⚡    Behavior"]
        C1[Instance Dict<br/>Takes Precedence]
        C2[Descriptor<br/>Takes Precedence]
        C3[Descriptor<br/>Takes Precedence]
        C4[Complete<br/>Control]
    end
    
    %% Connections
    A1 --> B1
    A2 --> B2
    A3 --> B3
    A4 --> B4
    B1 --> C1
    B2 --> C2
    B3 --> C3
    B4 --> C4
    
    %% Styling connections
    linkStyle 0 stroke:#1976d2,stroke-width:3px
    linkStyle 1 stroke:#1976d2,stroke-width:3px
    linkStyle 2 stroke:#1976d2,stroke-width:3px
    linkStyle 3 stroke:#1976d2,stroke-width:3px
    linkStyle 4 stroke:#7b1fa2,stroke-width:3px
    linkStyle 5 stroke:#7b1fa2,stroke-width:3px
    linkStyle 6 stroke:#7b1fa2,stroke-width:3px
    linkStyle 7 stroke:#7b1fa2,stroke-width:3px
    
    %% Styling subgraphs
    style TYPES fill:#e8f4fd,stroke:#1976d2,stroke-width:3px,color:#000
    style CLASSIFICATION fill:#f8f0ff,stroke:#7b1fa2,stroke-width:3px,color:#000
    style BEHAVIOR fill:#fff4e6,stroke:#f57c00,stroke-width:3px,color:#000
    
    %% Styling nodes
    style A1 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style A2 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style A3 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style A4 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style B1 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style B2 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style B3 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style B4 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style C1 fill:#fff8e1,stroke:#f57c00,stroke-width:3px,color:#000
    style C2 fill:#fff8e1,stroke:#f57c00,stroke-width:3px,color:#000
    style C3 fill:#fff8e1,stroke:#f57c00,stroke-width:3px,color:#000
    style C4 fill:#fff8e1,stroke:#f57c00,stroke-width:3px,color:#000
```

### 🔍 Data vs Non-Data Descriptors

| Type | Definition | Precedence | Common Uses |
|------|------------|------------|-------------|
| **Non-Data** | Only `__get__()` | Instance dict wins | Functions, methods |
| **Data** | Has `__set__()` or `__delete__()` | Descriptor wins | Properties, validators |

[↑ Back to TOC](#-table-of-contents)

---

<a id="simple-example-constant-descriptor"></a>
## 🎲 Simple Example: Constant Descriptor

Let's start with the simplest possible descriptor that always returns a constant value:

### 💻 Code Example

```python
class Ten:
    """A descriptor that always returns 10"""
    
    def __get__(self, obj, objtype=None):
        return 10

class A:
    x = 5        # Regular class attribute
    y = Ten()    # Descriptor instance

# Usage
a = A()
print(a.x)  # Output: 5 (normal attribute lookup)
print(a.y)  # Output: 10 (descriptor lookup)
```

### 🔄 How It Works

```mermaid
graph TB
    subgraph ACCESS ["🔍    Attribute    Access"]
        A1[a.x access]
        A2[a.y access]
    end
    
    subgraph LOOKUP ["🔎    Lookup    Process"]
        B1[Find in class dict<br/>'x': 5]
        B2[Find in class dict<br/>'y': Ten instance]
        B3[Detect __get__<br/>method]
    end
    
    subgraph EXECUTION ["⚙️    Execution"]
        C1[Return value<br/>directly]
        C2[Call Ten.__get__]
        C3[Compute result]
    end
    
    subgraph RESULT ["📊    Result"]
        D1[Returns: 5]
        D2[Returns: 10]
    end
    
    %% Flow connections
    A1 --> B1
    A2 --> B2
    B1 --> C1
    B2 --> B3
    B3 --> C2
    C1 --> D1
    C2 --> C3
    C3 --> D2
    
    %% Styling connections
    linkStyle 0 stroke:#1976d2,stroke-width:3px
    linkStyle 1 stroke:#1976d2,stroke-width:3px
    linkStyle 2 stroke:#7b1fa2,stroke-width:3px
    linkStyle 3 stroke:#7b1fa2,stroke-width:3px
    linkStyle 4 stroke:#388e3c,stroke-width:3px
    linkStyle 5 stroke:#3f51b5,stroke-width:4px
    linkStyle 6 stroke:#388e3c,stroke-width:3px
    linkStyle 7 stroke:#3f51b5,stroke-width:4px
    
    %% Styling subgraphs
    style ACCESS fill:#e8f4fd,stroke:#1976d2,stroke-width:3px,color:#000
    style LOOKUP fill:#f8f0ff,stroke:#7b1fa2,stroke-width:3px,color:#000
    style EXECUTION fill:#f0f8f0,stroke:#388e3c,stroke-width:3px,color:#000
    style RESULT fill:#fff4e6,stroke:#f57c00,stroke-width:3px,color:#000
    
    %% Styling nodes
    style A1 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style A2 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style B1 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style B2 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style B3 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style C1 fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000
    style C2 fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000
    style C3 fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000
    style D1 fill:#fff8e1,stroke:#f57c00,stroke-width:3px,color:#000
    style D2 fill:#fff8e1,stroke:#f57c00,stroke-width:3px,color:#000
```

### 📝 Key Insights

- ✅ The value `10` is **not stored** in any dictionary
- ✅ The value is **computed on demand** by calling `__get__()`
- ✅ The descriptor must be a **class variable**, not an instance variable
- ✅ This is a **non-data descriptor** (only has `__get__()`)

[↑ Back to TOC](#-table-of-contents)

---

<a id="dynamic-lookups"></a>
## 🔄 Dynamic Lookups

Descriptors become truly useful when they compute values dynamically instead of returning constants:

### 💻 Directory Size Example

```python
import os

class DirectorySize:
    """A descriptor that computes directory size on access"""
    
    def __get__(self, obj, objtype=None):
        return len(os.listdir(obj.dirname))

class Directory:
    size = DirectorySize()  # Descriptor instance
    
    def __init__(self, dirname):
        self.dirname = dirname  # Regular instance attribute

# Usage
songs = Directory('songs')
games = Directory('games')

print(songs.size)  # Output: 20 (computed dynamically)
print(games.size)  # Output: 3 (computed dynamically)

# Delete a file
os.remove('games/chess')
print(games.size)  # Output: 2 (automatically updated!)
```

### 🎯 Parameter Roles

Understanding what the `__get__` parameters represent:

```mermaid
graph TB
    subgraph CALL ["📞    Method    Call"]
        A1["DirectorySize.__get__<br/>(self, obj, objtype)"]
    end
    
    subgraph PARAMS ["📦    Parameter    Values"]
        B1["self = size<br/>(DirectorySize instance)"]
        B2["obj = games<br/>(Directory instance)"]
        B3["objtype = Directory<br/>(the class)"]
    end
    
    subgraph ACCESS ["🔑    What    It    Accesses"]
        C1["Descriptor's<br/>own data"]
        C2["Instance's<br/>attributes<br/>(obj.dirname)"]
        C3["Class-level<br/>information"]
    end
    
    subgraph USAGE ["💡    Typical    Usage"]
        D1["Descriptor<br/>configuration"]
        D2["Get data from<br/>target object"]
        D3["Introspection<br/>metadata"]
    end
    
    %% Connections
    A1 --> B1
    A1 --> B2
    A1 --> B3
    B1 --> C1
    B2 --> C2
    B3 --> C3
    C1 --> D1
    C2 --> D2
    C3 --> D3
    
    %% Styling connections
    linkStyle 0 stroke:#1976d2,stroke-width:3px
    linkStyle 1 stroke:#1976d2,stroke-width:3px
    linkStyle 2 stroke:#1976d2,stroke-width:3px
    linkStyle 3 stroke:#7b1fa2,stroke-width:3px
    linkStyle 4 stroke:#7b1fa2,stroke-width:3px
    linkStyle 5 stroke:#7b1fa2,stroke-width:3px
    linkStyle 6 stroke:#3f51b5,stroke-width:4px
    linkStyle 7 stroke:#3f51b5,stroke-width:4px
    linkStyle 8 stroke:#3f51b5,stroke-width:4px
    
    %% Styling subgraphs
    style CALL fill:#e8f4fd,stroke:#1976d2,stroke-width:3px,color:#000
    style PARAMS fill:#f8f0ff,stroke:#7b1fa2,stroke-width:3px,color:#000
    style ACCESS fill:#f0f8f0,stroke:#388e3c,stroke-width:3px,color:#000
    style USAGE fill:#fff4e6,stroke:#f57c00,stroke-width:3px,color:#000
    
    %% Styling nodes
    style A1 fill:#e3f2fd,stroke:#1976d2,stroke-width:3px,color:#000
    style B1 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style B2 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style B3 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style C1 fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000
    style C2 fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000
    style C3 fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000
    style D1 fill:#fff8e1,stroke:#f57c00,stroke-width:2px,color:#000
    style D2 fill:#fff8e1,stroke:#f57c00,stroke-width:2px,color:#000
    style D3 fill:#fff8e1,stroke:#f57c00,stroke-width:2px,color:#000
```

### 🌟 Why This Is Powerful

| Aspect | Benefit |
|--------|---------|
| 🔄 **Dynamic** | Values computed fresh each time |
| 🎯 **Targeted** | Different result for each instance |
| 🧩 **Reusable** | Same descriptor for multiple classes |
| 🔍 **Transparent** | Looks like normal attribute access |

[↑ Back to TOC](#-table-of-contents)

---

<a id="when-descriptors-are-invoked"></a>
## ⚡ When Descriptors Are Invoked

Descriptors are automatically invoked during the **dot operator** attribute access, but not through direct dictionary access:

### 🎭 Invocation Scenarios

```python
class MyDescriptor:
    def __get__(self, obj, objtype=None):
        print("__get__ called!")
        return 42

class MyClass:
    attr = MyDescriptor()

obj = MyClass()

# INVOKES descriptor
value = obj.attr  # Prints: __get__ called!

# DOES NOT INVOKE descriptor (returns descriptor instance)
descriptor_obj = vars(MyClass)['attr']
print(type(descriptor_obj))  # Output: <class 'MyDescriptor'>
```

### 🚦 Invocation Flow

```mermaid
graph TB
    Start[Attribute Access Request]
    
    subgraph access[" "]
        A1[Dot Operator obj.attr]
        A2[getattr obj attr default]
        A3[vars obj or obj.__dict__]
    end
    
    subgraph method_call[" "]
        M1[Calls type obj __getattribute__ obj attr]
        M2[Calls type obj __getattribute__ obj attr]
        M3[Returns obj.__dict__ then access key]
    end
    
    subgraph lookup_order[" "]
        L1[Step 1: Check type obj __dict__ for attr]
        L2[Step 2: Walk through MRO chain]
        L3[Step 3: Check if found object has __get__]
        L4[Step 4: Is it data descriptor has __set__ or __delete__]
        L5[Step 5: Check instance obj.__dict__]
        L6[Step 6: If descriptor found earlier call __get__]
        L7[Step 7: Return value from class or raise AttributeError]
    end
    
    subgraph descriptor_types[" "]
        DD[Data Descriptor: has __set__ or __delete__]
        ND[Non-Data Descriptor: only has __get__]
        RO[Regular Object: no descriptor methods]
    end
    
    subgraph invocation[" "]
        I1[Call descriptor.__get__ self obj type obj]
        I2[instance is obj owner is type obj]
        I3[If obj is None class access owner is class]
    end
    
    subgraph examples[" "]
        E1[property: data descriptor computes value]
        E2[method: non-data descriptor creates bound method]
        E3[classmethod: non-data descriptor binds to class]
        E4[staticmethod: non-data descriptor returns unwrapped]
    end
    
    subgraph dict_bypass[" "]
        DB1[Direct dictionary access NO protocol]
        DB2[Returns property object itself]
        DB3[Returns function object unbound]
        DB4[Returns descriptor instance raw]
        DB5[No __get__ invocation at all]
    end
    
    subgraph fallback[" "]
        F1[If AttributeError in __getattribute__]
        F2[Call __getattr__ if defined]
        F3[Otherwise raise AttributeError]
    end
    
    subgraph performance[" "]
        P1[Dot operator: optimized C code path]
        P2[getattr: Python function overhead then same]
        P3[dict access: fastest but breaks semantics]
    end
    
    Start --> A1
    Start --> A2
    Start --> A3
    
    A1 --> M1
    A2 --> M2
    A3 --> M3
    
    M1 --> L1
    M2 --> L1
    M3 --> DB1
    
    L1 --> L2
    L2 --> L3
    L3 --> L4
    
    L4 -->|Yes| DD
    L4 -->|No| L5
    
    DD --> I1
    
    L5 -->|Found| RO
    L5 -->|Not found| L6
    
    L6 --> ND
    ND --> I1
    
    L6 -->|No descriptor| L7
    
    I1 --> I2
    I2 --> I3
    
    I1 --> E1
    I1 --> E2
    I1 --> E3
    I1 --> E4
    
    M1 --> F1
    M2 --> F1
    F1 --> F2
    F2 --> F3
    
    DB1 --> DB2
    DB1 --> DB3
    DB1 --> DB4
    DB1 --> DB5
    
    A1 -.->|Performance| P1
    A2 -.->|Performance| P2
    A3 -.->|Performance| P3
    
    classDef access fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef method fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef lookup fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef descriptor fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef invoke fill:#f3e5f5,stroke:#6a1b9a,stroke-width:2px
    classDef example fill:#fff8e1,stroke:#f57c00,stroke-width:2px
    classDef bypass fill:#fce4ec,stroke:#c2185b,stroke-width:3px
    classDef fallback fill:#ffebee,stroke:#d32f2f,stroke-width:2px
    classDef perf fill:#e0f2f1,stroke:#00695c,stroke-width:2px
    
    class A1,A2,A3 access
    class M1,M2,M3 method
    class L1,L2,L3,L4,L5,L6,L7 lookup
    class DD,ND,RO descriptor
    class I1,I2,I3 invoke
    class E1,E2,E3,E4 example
    class DB1,DB2,DB3,DB4,DB5 bypass
    class F1,F2,F3 fallback
    class P1,P2,P3 perf
```

### ⚠️ Critical Rules

| Rule | Description |
|------|-------------|
| 📍 **Class Variable Only** | Descriptors must be stored as class attributes, not instance attributes |
| 🔵 **Dot Operator** | Descriptors are invoked by dot notation (`obj.attr`) |
| 🚫 **Dict Bypass** | Direct dictionary access returns the descriptor object itself |
| 🎯 **Instance Lookup** | `obj.attr` calls `descriptor.__get__(obj, type(obj))` |
| 🏛️ **Class Lookup** | `Class.attr` calls `descriptor.__get__(None, Class)` |

[↑ Back to TOC](#-table-of-contents)

---

<a id="key-concepts-summary"></a>
## 📚 Key Concepts Summary

### 🎯 Essential Takeaways

```mermaid
graph TB
    subgraph WHAT ["❓    What    Descriptors    Are"]
        A1[Objects with<br/>__get__, __set__,<br/>or __delete__]
        A2[Stored as<br/>class variables]
        A3[Control attribute<br/>access behavior]
    end
    
    subgraph WHY ["💡    Why    Use    Them"]
        B1[Add custom logic<br/>to attributes]
        B2[Reusable attribute<br/>behaviors]
        B3[Foundation for<br/>properties & methods]
    end
    
    subgraph HOW ["⚙️    How    They    Work"]
        C1[Invoked by<br/>dot operator]
        C2[Receive obj & type<br/>as parameters]
        C3[Can compute values<br/>dynamically]
    end
    
    subgraph TYPES ["🏷️    Descriptor    Types"]
        D1[Non-Data:<br/>Only __get__]
        D2[Data:<br/>Has __set__/__delete__]
    end
    
    subgraph CENTER ["🎯    Python    Descriptors"]
        E1[Core Feature]
    end
    
    %% Connections to center
    A1 --> E1
    A2 --> E1
    A3 --> E1
    B1 --> E1
    B2 --> E1
    B3 --> E1
    C1 --> E1
    C2 --> E1
    C3 --> E1
    D1 --> E1
    D2 --> E1
    
    %% Styling connections
    linkStyle 0,1,2 stroke:#1976d2,stroke-width:3px
    linkStyle 3,4,5 stroke:#7b1fa2,stroke-width:3px
    linkStyle 6,7,8 stroke:#388e3c,stroke-width:3px
    linkStyle 9,10 stroke:#f57c00,stroke-width:3px
    
    %% Styling subgraphs
    style WHAT fill:#e8f4fd,stroke:#1976d2,stroke-width:3px,color:#000
    style WHY fill:#f8f0ff,stroke:#7b1fa2,stroke-width:3px,color:#000
    style HOW fill:#f0f8f0,stroke:#388e3c,stroke-width:3px,color:#000
    style TYPES fill:#fff4e6,stroke:#f57c00,stroke-width:3px,color:#000
    style CENTER fill:#e8eaf6,stroke:#3f51b5,stroke-width:4px,color:#000
    
    %% Styling nodes
    style A1 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style A2 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style A3 fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style B1 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style B2 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style B3 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style C1 fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000
    style C2 fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000
    style C3 fill:#e8f5e8,stroke:#388e3c,stroke-width:2px,color:#000
    style D1 fill:#fff8e1,stroke:#f57c00,stroke-width:2px,color:#000
    style D2 fill:#fff8e1,stroke:#f57c00,stroke-width:2px,color:#000
    style E1 fill:#e8eaf6,stroke:#3f51b5,stroke-width:4px,color:#000
```

### 📋 Quick Reference Card

| Concept | Definition | Example |
|---------|------------|---------|
| **Descriptor** | Object with `__get__`, `__set__`, or `__delete__` | `class MyDescriptor: def __get__(...): ...` |
| **Data Descriptor** | Has `__set__` or `__delete__` | Properties, validators |
| **Non-Data Descriptor** | Only has `__get__` | Functions, methods |
| **Invocation** | Via dot operator on class variables | `obj.attr` triggers `__get__` |
| **Parameters** | `(self, obj, objtype)` | Access descriptor, instance, class |

### 🎓 What You've Learned

✅ **Descriptors** are objects that customize attribute access  
✅ They use special methods: `__get__`, `__set__`, `__delete__`  
✅ They must be **class variables** to work  
✅ **Dynamic lookups** compute values on-demand  
✅ The **dot operator** invokes descriptors automatically  
✅ **Data descriptors** override instance dictionaries  
✅ **Non-data descriptors** are overridden by instance attributes  

[↑ Back to TOC](#-table-of-contents)

---

## 🎯 Next Steps

Ready to dive deeper? In **Section 2: Descriptor Protocol Deep Dive**, we'll explore:

- 🔬 The complete attribute lookup chain
- 🎛️ Detailed mechanics of `__get__`, `__set__`, and `__delete__`
- 🔄 How data vs non-data descriptors differ in practice
- 🏗️ The `__set_name__` method for automatic configuration
- 🔍 Instance vs class vs super() invocation patterns

---

**📖 End of Section 1**

*Continue to Section 2 for advanced descriptor mechanics and implementation details.*
