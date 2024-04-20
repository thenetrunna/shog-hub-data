# Shog Hub Data

This repository contains data used by [Shog Hub](https://hub.shog.ai) to track Shoggoth resources across all nodes.


## Adding new resources

Follow the below steps to add a new resource:

### Add namesapce image

Add a 200x200 image to ./users 

### Create namespace directory

Ensure the namespace has a directory in ./resources . if the directory does not exist, create it.

### Create resource JSON file

Create a json file with the name of the resource in the namespace's directory.

Edit the file and add the relevant details using the bellow example:

```json
{
  "shoggoth_id": "SHOG9d7196366c54a076122f4f1e906339a0a0ec1a9397377002c72d9f44aef4fe15",
  "namespace": "karpathy",
  "resource": "llm.c",
  "label": "llm.c-6b49ed1.zip",
  "is_code": true,
  "size": "83 KB"
}
```
