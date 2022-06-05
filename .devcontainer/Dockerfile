FROM jupyter/scipy-notebook:44fe185f9495
# pandas、numpy、scipy、 scikit-image、matplotlibなど一通り入っているjupyter/scipy-notebookを使う。
# また、常にpullするimageを固定したいので、一意のtagをつける。
USER jovyan
RUN conda install -y jupyter_contrib_nbextensions