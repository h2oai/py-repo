# Steps for building new pyarrow wheels

On a PPC machine run the following docker script:
```
mkdir -p wheels 
cd wheels
docker run --rm --init -u `id -u`:`id -g` -v `pwd`:/dot -e HOME=/tmp -it quay.io/pypa/manylinux2014_ppc64le

cd ~
/opt/python/cp37-cp37m/bin/pip install virtualenv
/opt/python/cp37-cp37m/bin/python -m virtualenv py37
source py37/bin/activate
export PS1="\n\033[38;5;34m(`basename $VIRTUAL_ENV`) [docker:$AUDITWHEEL_PLAT] \033[38;5;222m\w # \033[m"
export ARROW_HOME=/tmp/local/arrow
mkdir -p $ARROW_HOME

pip install pip --upgrade
pip install cython
pip install -i https://h2oai.github.io/py-repo/ numpy

git clone https://github.com/apache/arrow
cd arrow/cpp
cmake -DCMAKE_INSTALL_PREFIX=$ARROW_HOME -DARROW_CXXFLAGS="-lutil" -DARROW_PYTHON=on -DARROW_ORC=on -DARROW_PARQUET=on
make -j4 
make install

cd ../python
python setup.py build_ext --build-type=release --bundle-arrow-cpp bdist_wheel
cp dist/*.whl /dot
```

Then upload these wheel files to S3:
```
s3cmd put pyarrow*.whl s3://artifacts.h2o.ai/releases/ai/h2o/thirdparty/pyarrow/ --acl-public
```

Finally, run the following python script to generate links that should be added to this repo's `numpy/index.html` file:
```
import glob
import hashlib
for filename in glob.glob("pyarrow*.whl"):
  sha = hashlib.sha256(open(filename, 'rb').read()).hexdigest()
  print('    <li><a href="http://artifacts.h2o.ai.s3.amazonaws.com/releases/'
        'ai/h2o/thirdparty/pyarrow/%s#sha256=%s">%s</a></li>'
        % (filename, sha, filename))
```
