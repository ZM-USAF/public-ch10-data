# public-ch10-data
Repo containing information regarding publicly available chapter 10 data

## Public NASA ch10 data

The data derived from NASA Tail 667 flights is stored in the bucket `nasa-public-ch10`. It has the following structure:

```
chapter10/*.c10
ICD/icd.yaml
parsed/*.parquet
translated/*.parquet
```

This bucket is set to `request-pays`, so that data access costs will be incurred by the requester of the data. Because of this, an AWS account with public/private access keys is required. See [these docs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RequesterPaysBuckets.html) for more information.

## Accessing the NASA ch10 data

The following assumes that python is installed with the latest versions of s3fs, pandas, and pyarrow. To install these via conda, please run the following from the root of this repository:

```
conda env create -f environment.yaml
```

With these packages installed s3fs filesystem can then be instantiated. Please replace `<public_key>` and `<private_key>` with the appropriate keys from AWS:

```python
import s3fs

fs = s3fs.S3FileSystem(
    key='<public_key>', 
    secret='<private_key>', 
    anon=False, 
    requester_pays=True
)
```

To list out the available files in the bucket, run:

```python
top_keys = fs.ls('s3://nasa-public-ch10/')
print(top_keys)

['nasa-public-ch10/ICD',
 'nasa-public-ch10/chapter10',
 'nasa-public-ch10/parsed',
 'nasa-public-ch10/translated']
```

This will show the 4 top level keys in the bucket. Running the same command with the subkeys such as `fs.ls('s3://nasa-public-ch10/parsed/') will display the filenames of the data.

## Downloading and displaying data

An example of downloading a chapter 10 file to the `/tmp` directory locally can be seen below:

```python
fs.get('s3://nasa-public-ch10/chapter10/667200103210556.ch10', '/tmp/667200103210556.ch10')
```

While the ch10 data is binary, the `parsed` and `translated` data are both in parquest and can be read in with pandas:

```
import pyarrow.parquet as pq
import pandas as pd

parsed_example = (
    pq.ParquetDataset(
    'nasa-public-ch10/translated/667200103210556_1553_translated_NAV__00.parquet', 
    filesystem=fs)
    .read_pandas()
    .to_pandas()
)

translated_example = (
    pq.ParquetDataset(
    's3://nasa-public-ch10/translated/667200103210556_1553_translated_NAV__00.parquet', 
    filesystem=fs)
    .read_pandas()
    .to_pandas()
)
```
