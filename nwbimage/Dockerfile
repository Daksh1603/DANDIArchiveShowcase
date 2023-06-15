FROM node:15 as clone
ENV BRANCH_TAG="development"
ENV REPO=https://github.com/MetaCell/nwb-explorer.git
RUN echo "cache 2022-09-30-a"
RUN git clone $REPO -b $BRANCH_TAG
RUN rm -Rf .git

FROM node:15 as jsbuild


ENV FOLDER=nwb-explorer



WORKDIR $FOLDER/webapp

COPY --from=clone nwb-explorer/webapp/package.json .
COPY --from=clone nwb-explorer/webapp/package-lock.json .
RUN npm ci
COPY --from=clone nwb-explorer/webapp/ .

RUN npm run build

RUN mv nwb-explorer/node_modules/@geppettoengine .
RUN rm nwb-explorer/-Rf node_modules/*
RUN mv nwb-explorer/@geppettoengine node_modules

###
FROM jupyter/base-notebook:hub-1.4.2

ENV FOLDER=nwb-explorer
USER root

#


RUN jupyter labextension disable @jupyterlab/hub-extension
RUN rm -rf /var/lib/apt/lists

RUN apt-get update -qq &&\
    apt-get install python3-tk vim nano unzip git g++ libjpeg-dev zlib1g-dev -qq
    


RUN chown $NB_UID /opt
RUN chown $NB_UID .




COPY --from=clone $FOLDER/requirements.txt requirements.txt
RUN pip install -r requirements.txt --no-cache-dir

USER $NB_UID
COPY --from=clone $FOLDER .
COPY --from=jsbuild --chown=1000:1000 $FOLDER $FOLDER


WORKDIR $FOLDER

# Update Browserslist
RUN cd webapp && npx browserslist@latest --update-db

RUN cd ../

RUN pip install setuptools==45

USER root
RUN python utilities/install.py 

RUN chown -R $NB_UID .
RUN chown -R $NB_UID /opt/*


USER $NB_UID
CMD ./NWBE
