# Steps for building new pandas wheels

On a PPC machine run the following docker command (note: this may take up to an hour):
```bash
mkdir -p wheels 
cd wheels
docker run --rm --init -u `id -u`:`id -g` --entrypoint /bin/bash -v `pwd`:/dot -e HOME=/tmp \
    quay.io/pypa/manylinux2014_ppc64le \
    -c "cd /tmp && \
        /opt/python/cp36-cp36m/bin/python -m pip install --upgrade --user pip && \
        /opt/python/cp37-cp37m/bin/python -m pip install --upgrade --user pip && \
        /opt/python/cp38-cp38/bin/python  -m pip install --upgrade --user pip && \
        /opt/python/cp36-cp36m/bin/python -m pip wheel pandas && \
        /opt/python/cp37-cp37m/bin/python -m pip wheel pandas && \
        /opt/python/cp38-cp38/bin/python  -m pip wheel pandas && \
        ls -la && \
        mv *.whl /dot"
```
Pandas doesn't support python3.5 starting with version `1.0.0`.

Then upload these wheel files to S3:
```
s3cmd put pandas*.whl s3://artifacts.h2o.ai/releases/ai/h2o/thirdparty/pandas/ --acl-public
```

Finally, run the following python script to generate links that should be added to this repo's `pandas/index.html` file:
```python
import glob
import hashlib
for filename in glob.glob("pandas*.whl"):
  sha = hashlib.sha256(open(filename, 'rb').read()).hexdigest()
  print('    <li><a href="http://artifacts.h2o.ai.s3.amazonaws.com/releases/'
        'ai/h2o/thirdparty/pandas/%s#sha256=%s">%s</a></li>'
        % (filename, sha, filename))
```
