# Weather Radar Reflectivity Dataset - KJAX NEXRAD 2021

With the challenges posed by climate change, rain forecasting becomes increasingly important. Doppler-based weather radars are used to measure atmospheric reflectivity that indicates the presence of precipitation, hail, snow, or smoke. Machine learning techniques have been used successfully in several areas. Therefore, the objective of this paper is to offer a dataset of sequences of reflectance images composed of 4097 sequences of 20 images, totaling 81,940 samples that can be used to train of neural networks. The raw data used in the generation of the proposed set were collected by the NEXRAD weather radar system during the year 2021 in the state of Florida, USA. An example of how this dataset can be used is in the training of neural networks that provide a short-term precipitation forecast, a technique known as nowcasting.

## Paper
Coming soon.

## Ready to use dataset
Due to GitHub hosting size restrictions, the dataset is available for download upon request. Please, contact me if you want access to the Google Drive file.

## Compiling from source

### 1. Download Raw Data
First step is to download data from a Open Access provider ([AWS](https://registry.opendata.aws/noaa-nexrad/), [GCP](https://cloud.google.com/storage/docs/public-datasets/nexrad)). This raw data is composed of binary files containing [Level 2 NEXRAD](https://www.ncei.noaa.gov/access/metadata/landing-page/bin/iso?id=gov.noaa.ncdc:C00345) products that have to be further processed into a reflectivity picture.


```python
!clone copy GCP:gcp-public-data-nexrad-l2 --include "/2021/*/*/KJAX/*" . -P
```

### 2. Extract
The data downloaded in the previous step is compressed. The following command is used to locate all files and decompress them.

```python
!find . -name '*.tar' -execdir tar -xvf '{}' \;
```

### 3. Process
The extracted data is in the Archive 2 (`.ar2v`). Each file contains information about one NEXRAD observation. To render this data into a precipitation product, the program [`go-nexrad`](https://github.com/bwiggs/go-nexrad) will be used. Follow the steps at the repository to download and install this script. To process all files, the following command is used.

```python
!find . -name "*ar2v" | parallel -I% --max-args 1 nexrad-render % -o %.png -s 2048
```

### 4. Post-processing

```python
import os 

# Remove `.ar2v` and replace with `_REF` (reflectivity).
ifiles = os.listdir()

for inf in ifiles:
    ouf = inf.replace(".ar2v", "_REF")
    os.replace(inf, ouf)

# Group frames together.
def divide_chunks(l, n):
    for i in range(0, len(l), n):
        yield l[i:i + n]

chunks = list(divide_chunks(sorted(os.listdir()), 20))

for i, chunk in enumerate(chunks):
    os.mkdir(str(i))
    for file in chunk:
        os.replace(file, os.path.join(str(i), file)) 
```
