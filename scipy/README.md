# Steps for building new numpy wheels

On a PPC machine run the following docker command (note: this may take up to an hour):
```
mkdir -p wheels 
cd wheels
docker run --rm --init --entrypoint /bin/bash -v `pwd`:/dot -e HOME=/tmp \
    quay.io/pypa/manylinux2014_ppc64le \
    -c "yum update -y && yum install -y lapack-devel lapack64-devel && \
        cd /tmp && \
        /opt/python/cp36-cp36m/bin/python -m pip wheel scipy && \
        /opt/python/cp37-cp37m/bin/python -m pip wheel scipy && \
        /opt/python/cp38-cp38/bin/python  -m pip wheel scipy && \
        ls -la && \
        mv *.whl /dot"
```

Then upload these wheel files to S3:
```
s3cmd put scipy*.whl s3://artifacts.h2o.ai/releases/ai/h2o/thirdparty/scipy/ --acl-public
```

Finally, run the following python script to generate links that should be added to this repo's `scipy/index.html` file:
```
import glob
import hashlib
for filename in glob.glob("scipy*.whl"):
  sha = hashlib.sha256(open(filename, 'rb').read()).hexdigest()
  print('    <li><a href="http://artifacts.h2o.ai.s3.amazonaws.com/releases/'
        'ai/h2o/thirdparty/scipy/%s#sha256=%s">%s</a></li>'
        % (filename, sha, filename))
```
