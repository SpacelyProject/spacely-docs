# Spacely Experiments and DataFiles

**Experiments** in Spacely are a data structure you can use to organize the data you take from experiments and keep track of metadata. 
- An **Experiment** object is a collection of **DataFiles** and **metadata** represented as key-value pairs.
- Experiment metadata can be accessed with **Experiment.set(key,value)** and **Experiment.get(key)**
- A DataFile object represents an ordinary file (for example a CSV), and you can write lines of data to it with **DataFile.write()**
- DataFiles can also have their own metadata, which can be accessed with **DataFile.set(key,value)** and **DataFile.get(key)**. 
- Experiment metadata is considered to be the “default” values, while DataFile metadata contains anything that varies from data file to data file. If DataFile does not have metadata for “x”, then DataFile.get(x) will return Experiment.get(x)

This pseudo-code demonstrates how DataFile metadata overrides Experiment metadata:
```
DataFile.get(x):
	if DataFile.metadata[x] exists:
 return DataFile.metadata[x]
	else: 
		return Experiment.get(x)
```
