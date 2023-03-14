# Embed interactive itkwidgets 3D renderings into JupyterLite deployments 

###### tags: `post`, `draft`, `itk`, `itkwidgets`, `jupyter`, `jupyterlite`

###### By: Matt McCormick [![ORCID](https://info.orcid.org/wp-content/uploads/2020/12/orcid_16x16.gif)](https://orcid.org/0000-0001-9475-3756), Brianna Major [![ORCID](https://info.orcid.org/wp-content/uploads/2020/12/orcid_16x16.gif)](https://orcid.org/0000-0003-4968-5701), Jeremy Tuloup [![ORCID](https://info.orcid.org/wp-content/uploads/2020/12/orcid_16x16.gif)](), Wei Ouyang [![ORCID](https://info.orcid.org/wp-content/uploads/2020/12/orcid_16x16.gif)](https://orcid.org/0000-0002-0291-926x), Stephen Aylward [![ORCID](https://info.orcid.org/wp-content/uploads/2020/12/orcid_16x16.gif)](https://orcid.org/0000-0002-7862-8856)

**Zero-install** web applications have transformed the way we consume and deliver software. Browser-based interfaces facilitate rapid discovery, exploration, and universal access.

However, for research software engineers (RSEs), developing traditional software stacks for web applications is not only onerous, but those stacks may limit essential future scalability and may be even more onerous to sustain. A RSE must face difficult questions:

- Who is going to pay to keep the servers online?
- Who is going to pay to scale the servers for many user or datasets?
- When are you going to find the time to learn and keep up-to-date with all the devops knowledge and skills required?
- Who is going to maintain the system and address security vulnerabilities as they arise?
- How is private data on the server managed and kept secure?

As one of my favorite professors used to say, in cases like this we can look to the advice offered by a wise doctor:

> Patient: Oh, Doctor, it hurts badly when I move my knee like this.
> 
> Doctor: Stop moving your knee like that!

In some cases, components of the traditional web application software stack are necessary, and some of those components are easier, more scalable, and more sustainable than others. However, for many RSE use cases, we now can create useful web applications while avoiding traditional server-related hardships altogether.

In this tutorial, we will demonstrate how to create a **zero-server** JupyterLite deployment that embeds interactive 3D renderings into advanced scientific applications, such as for deep learning medical image analysis applications using [MONAI](https://monai.io). 

[JupyterLite](https://jupyterlite.readthedocs.io/en/latest/) is a [JupyterLab](https://jupyter.org) distribution that runs entirely in the browser built from the ground-up using JupyterLab components and extensions. JupyterLite uses a [WebAssembly](webassembly.org)-based distribution of scientific Python called [Pyodide](https://pyodide.org/en/stable/).

[ITKWidgets](https://itkwidgets.readthedocs.io/) provides interactive widgets to visualize images, point sets, and 3D geometry on the web. ITKWidgets is powered by the same WebAssembly technology. It is built on [ITK-Wasm](https://wasm.itk.org) and [ImJoy](https://imjoy.io/), a hybrid computing platform that communicates via symmetrical transparent remote procedure calls. ImJoy and ITKWidgets support browser-based Pyodide communication along with a number of additional server-client communication transport mechanisms.

In this tutorial, we will first demonstrate a zero-server, interactive 3D rendering notebook. Then, we walk through the quick and easy configuration that can be customized to your needs.  Let's get started! 🚀

## 0. Preliminaries

Reproduce the figure below, a rendering of medical imaging volume of an abdominal aortic stent, by [running the notebook in your web browser](https://jupyterlite-itkwidgets-config-post.netlify.app/lab/index.html?path=Hello3DWorld.ipynb)! After the page has loaded, use the standard `Shift+Enter` keys to execute the Jupyter notebook cells.

![a medical imaging volume of an abdominal aortic stent rendered in JupyterLite](https://i.imgur.com/8VilXUN.gif)

Note that unlike other Jupyter deployments, the python code runs on your system instead of a server.

## 1. Create the JupyterLite environment

To build our sustainable JupyterLite deployment, we will use [a Python environment](https://github.com/conda-forge/miniforge) that contains Python packages for:

1. `jupyterlite`
2. `imjoy_jupyterlab_extension`
3. Any other JupyterLab federated extensions (JupyterLab 3 extensions) that you want in your JupyterLab deployment.

Create a *requirements.txt* file with:

```
jupyterlite[all]==0.1.0b17
imjoy_jupyterlab_extension
```

And install the packages:

```shell
python -m pip install -r ./requirements.txt
```

See also the related [JupyterLite extension addition documentation](https://jupyterlite.readthedocs.io/en/latest/howto/configure/simple_extensions.html).

## 2. Add itkwidgets and other Python packages

Next, we will add *itkwidgets*, its dependencies, and other Python packages and their dependencies, that we wish to include into the JupyterLite configuration for deployment. These packages, along with [the packages available in the Pyodide distribution](https://github.com/pyodide/pyodide/tree/main/packages), will be available in the deployed site.

Create a *jupyterlite_config.json* file, which specifies the locations of the itkwidgets wheel Python packages. Add other desired packages and their dependencies as follows.

```js 
{
    "PipliteAddon": {
        "piplite_urls": [
            "https://files.pythonhosted.org/packages/4c/ee/56f970ca26375176d3e4885f58471a12d5a6794bcefe8ad0ccb8d7158ca3/itkwasm-1.0b82-py3-none-any.whl",
            "https://files.pythonhosted.org/packages/6c/55/c3fc7e2b9671d15f0c0becdcb9fad6c330172988744ad6eaa17b71bace88/imjoy_rpc-0.5.16-py3-none-any.whl",
            "https://files.pythonhosted.org/packages/69/d9/5a6c8af2f4b4f49a809ae316ae4c12937d7dfda4e5b2f9e4167df5f15c0e/imjoy_utils-0.1.2-py3-none-any.whl",
            "https://files.pythonhosted.org/packages/c9/dc/3504845528418aff0b71f4b622bb0e8e12adec2d8f2c1ba21d695b9ac6e6/itkwidgets-1.0a24-py3-none-any.whl",
            "https://files.pythonhosted.org/packages/bb/3e/3667ac685ae83887b874896bcb55584797ba6b52a292df3e4b37736a9610/ngff_zarr-0.1.6-py3-none-any.whl"
        ]
    }
}
```

You can find links to these URLs by browsing the package on [PyPI](https://pypi.org) and copying the link from the *Download files* page for a package.

For packages that do not have a wheel on PyPI, you can provide one locally by placing them in the *pypi/* directory of your JupyterLite configuration. For example, if you want to use a local version of itkwidgets instead of the version on PyPi, you could directly add [this `dask-image` wheel](https://github.com/InsightSoftwareConsortium/itkwidgets/raw/2baa8ec865d4c08a4749cc468579742448e524c7/docs/jupyterlite/pypi/dask_image-2022.9.0-py2.py3-none-any.whl) to your *pypi/* directory.

```sh
$ ls pypi/
pypi/dask_image-2022.9.0-py2.py3-none-any.whl
```

## 3. Add notebooks and data

Add notebooks and data you would like available in the deployment in the *files/* directory. In this tutorial, we will add a [*Hello3DWorld.ipynb* notebook](https://github.com/InsightSoftwareConsortium/itkwidgets/raw/2baa8ec865d4c08a4749cc468579742448e524c7/docs/jupyterlite/files/Hello3DWorld.ipynb).

```sh
$ ls files/
files/Hello3DWorld.ipynb
```

In your notebook, install additional packages in the first cell with `piplite`.

```python
import piplite
await piplite.install(["itkwidgets==1.0a24"])
```

## 4. Build and deploy

Build your site with the command:

```sh
jupyter lite build
```

Serve the site locally with `python -m http.server --directory ./_output` or:

```sh
jupyter lite serve
```

The site can be deployed and shared with free static file hosting services such as [GitHub Pages](https://jupyterlite.readthedocs.io/en/latest/quickstart/deploy.html), [Netlify](https://jupyterlite.readthedocs.io/en/latest/howto/deployment/vercel-netlify.html), or [Fleek](https://fleek.co/).

## What's Next

In this post, we learned how to create a scalable, sustainable, zero-server Jupyter deployment that uses ITKWidgets for 3D rendering. In subsequent posts, we will discuss how to create simple, zero-server, custom web applications written in Python with [PyScript](https://pyscript.net/).

**Enjoy ITK!**
