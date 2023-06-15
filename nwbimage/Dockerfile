FROM node:15 as clone
ENV BRANCH_TAG="osb2"
ENV REPO=https://github.com/MetaCell/nwb-explorer.git
RUN git clone $REPO -b $BRANCH_TAG

FROM node:15 as jsbuild


ENV FOLDER=nwb-explorer



WORKDIR $FOLDER/webapp

COPY --from=clone nwb-explorer/webapp/package.json .
COPY --from=clone nwb-explorer/webapp/package-lock.json .
RUN npm ci
COPY --from=clone nwb-explorer/webapp/ .
COPY geppetto/GeppettoConfiguration.json GeppettoConfiguration.json
RUN npm run build


###
FROM jupyter/base-notebook:hub-1.4.2
ENV FOLDER=nwb-explorer
USER root

#

WORKDIR $FOLDER

RUN rm -rf /var/lib/apt/lists
RUN apt-get update -qq &&\
    apt-get install python3-tk vim nano unzip git g++ libjpeg-dev zlib1g-dev -qq
RUN pip install cython --no-cache-dir

RUN chown $NB_UID /opt
RUN chown $NB_UID .



USER $NB_UID

COPY --from=clone $FOLDER/requirements.txt requirements.txt
RUN pip install -r requirements.txt --no-cache-dir
COPY --from=clone $FOLDER .



RUN python utilities/install.py --npm-skip --no-test
COPY --from=clone $FOLDER/utilities/custom.css /home/jovyan/.jupyter/custom/custom.css

# this removes the frame ancestor default cors settings
RUN rm -f ~/.jupyter/*.json

USER root
# sym link workspace pvc to $FOLDER
RUN chown -R $NB_UID /opt/conda/etc/jupyter/nbconfig


RUN chown $NB_UID .


USER $NB_UID
CMD ./NWBE
