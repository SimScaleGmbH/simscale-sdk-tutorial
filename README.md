# SimScale SDK Tutorial

Sources for the public SimScale SDK Tutorial.

The generated tutorial website can be found in the following address:

[https://simscalegmbh.github.io/simscale-sdk-tutorial/](https://simscalegmbh.github.io/simscale-sdk-tutorial/)

## Development

This project is developed using the [Sphinx](https://www.sphinx-doc.org/en/master/index.html) documentation tool.

In order to build the HTML locally, first setup your dev environment:

1. Clone the repo and enter the folder

```bash
git clone https://github.com/SimScaleGmbH/simscale-sdk-tutorial.git

cd simscale-sdk-tutorial
```

2. Create and activate a virtual environment, in this case using [virtualenv](https://virtualenv.pypa.io/en/latest/)

```bash
virtualenv venv

. venv/bin/activate
```

3. Install the prerequisites

```bash
pip install -r requirements.txt
```

Now you are ready to build the MTHL page locally:

```bash
make html
```

You will find the generated website in the `_build/html/` folder. If you open the `index.html` file located inside that folder with your browser, you should be able to visualize the generated website.

## Contribute

If you wish to contribute, please feel free to submit a Pull Request or create an Issue.

The content of the tutorial is added in [reStructuredText](https://sublime-and-sphinx-guide.readthedocs.io/en/latest/index.html) format, in the corresponding `.rst` files, found inside the `tutorial` and `advanced` folders.