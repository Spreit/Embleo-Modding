# Manifest
Manifest contains a list of AssetBundles accessible to the game and their dependencies between each other.

Updating manifest is necessary after each AssetBundle edit, as the game performs checksum checks,
which changes with each edit, and the game won't re-download a file, if its entry parameters weren't changed.

###	m_Entries
List of AssetBundles that the game can access and therefore download from the **Asset Server**.

| Field            | Meaning                                                                   |
|------------------|---------------------------------------------------------------------------|
| **Name**         | AssetBundle path without **Version** string and `.ab` extension.          |
| **Version**      | 32-character hexadecimal string.                                          |
| **Crc**          | **CRC32** number. **Must** be recalculated after each AssetBundle change. |
| **Size**         | Size of the AssetBundle file, in bytes.                                   |
| **DependsIndex** | Reference to **m_Depends** dependency list (see below).                   |

Entries are stored alphabetically, however the correct sorting differs from standard Python `sort()`. 
Custom `manifest_sorting` function is used instead.
**Name** (AssetBundle path) starts from the root folder of the *Asset Server*, it's not container path that is shown in AssetStudio.

Actual manifest file needs to follow a naming convention: "Name_Version.ab"

Manifest file itself is not listed in **m_Entries**.

### m_Depends
**m_Depends** is a flat array, which contains lengths and **m_Entries** indexes.

**DependsIndex** refers to a length number in m_Depends, numbers after that (length amount of times)
are **m_Entries** indexes that original entry depends on.

### Example

```
...
"m_Entries": [
    {
      "Name": "a",
      "Version": "0123456789abcdef0123456789abcdef",
      "Crc": 478902324,
      "Size": 500,
      "DependsIndex": 0
    },
    {
      "Name": "b",
      "Version": "0123456789abcdef0123456789abcdef",
      "Crc": 3712333,
      "Size": 1024,
      "DependsIndex": 1
    },
    {
      "Name": "c",
      "Version": "0123456789abcdef0123456789abcdef",
      "Crc": 2789,
      "Size": 128,
      "DependsIndex": 3
    },
],
"m_Depends": [
    0,  
    1,  
    0
    2,  
    0,
    1
]
```

Unflattened **m_Depends** would look like:

```
[
    [], # A dependencies (none)
    [0], # B dependencies
    [0,1] # C dependencies
]
```

In the example, entry with **Name** `a` has **DependsIndex** of 0, which points to 0 in **m_Depends** array.
This 0 is the amount of entries `a` depends on, since it's the amount is 0, it doesn't depend on other AssetBundles.

`b`'s **DependsIndex** is 1, which also points to 1 in **m_Depends**. The next single number is an index of `a`, which `b` depends on.

However, `c` has **DependsIndex** of 3, since it has to point to a length, which in this case is 2 and the next 2 numbers are 0 and 1,
which means that `c` depends on both `a` and `b`.