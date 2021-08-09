# Custom JupyterHub UI for K8s

Basic UI customization for the JupyterHub.

## Local Configuration

1. (Recommended) Create and activate `virtualenv`:

    virtualenv -p python3 venv

2. Install JupyterHub and the Jupyter Notebook:

    python3 -m pip install jupyterhub notebook

3. Update the JupyterHub configruation file so that the `template_paths` config has the correct location for the custom templates provided in this repo.

> **NOTE**: the `jupyterhub_config.py` provided in this repo uses the standard `/usr/local/share/jupyterhub/...` location. If you installed the JupyterHub in your virtual environment then these locations are available in the `/path/to/venv/share/jupyterhub/...`.

4. Copy the custom CSS file to your JupyterHub css location. For example, if you installed the JupyterHub in your `virtualenv` then the location is most likely `/path/to/venv/share/jupyterhub/static/css/`.

> **TIP**: on Linux / MacOS systems, you can find the location of the CSS with `find $HOME/jhubui -iname style.min.css`.

Start JupyterHub with the example configuration file:

    jupyterhub -f jupyterhub_config.py

## Kubernetes Configuration

[These configuration instructions](https://discourse.jupyter.org/t/customizing-jupyterhub-on-kubernetes/1769/3) provide a nice alternative to baking the custom templates within the image.

Append the config below to your standard config to use the custom templates provided with this repo:

```python
jupyterhub:
  hub:
    # clone custom JupyterHub templates into a volume
    initContainers:
      - name: git-clone-templates
        image: alpine/git
        args:
          - clone
          - --single-branch
          - --branch=main
          - --depth=1
          - --
          - https://github.com/illumidesk/jhubui.git
          - /etc/jupyterhub/custom
        securityContext:
          runAsUser: 0
        volumeMounts:
          - name: custom-templates
            mountPath: /etc/jupyterhub/custom
    extraVolumes:
      - name: custom-templates
        emptyDir: {}
    extraVolumeMounts:
      - name: custom-templates
        mountPath: /etc/jupyterhub/custom

    extraConfig:
      templates: |
        c.JupyterHub.template_paths = ['/etc/jupyterhub/custom/templates']
```

Lemon squeezy ;-)

## License

MIT
