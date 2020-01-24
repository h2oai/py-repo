This repository satisfies [PEP-503](https://www.python.org/dev/peps/pep-0503/)
standard.

Its purpose is to provide pre-built binary files for some packages that we use 
but whose wheels were not published to PyPI.

## Usage ##
```
pip install -i https://h2oai.github.io/py-repo/ --extra-index-url https://pypi.org/simple/ packages...
```

## Adding new package ##

1. Create link in the main `index.html` file;
2. Create file `{PACKAGE}/index.html` if it doesn't exist yet;
3. In the file `{PACKAGE}/index.html` add the following line:
   `<a href="{URL}#sha256={SHA}>{FILENAME}</a>`, where
   - `{URL}` is a publicly available url of the binary file (which could be hosted on S3 for example);
   - `{SHA}` is obtained from python script 
     `import hashlib; print(hashlib.sha256(open("{FILENAME}", "rb").read()).hexdigest())`;
   - and `{FILENAME}` is the name of the wheel file.
