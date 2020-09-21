FROM drugilsberg/rdkit-ubuntu:latest
RUN apt-get update && apt-get install -y git
WORKDIR /paccmann_sarsvoc2/code
RUN git clone --branch sarscov2 https://github.com/PaccMann/paccmann_predictor 
RUN git clone --branch 0.0.2 https://github.com/PaccMann/toxsmi
RUN git clone --branch sarscov2 https://github.com/PaccMann/paccmann_omics 
RUN git clone --branch sarscov2 https://github.com/PaccMann/paccmann_chemistry 
RUN git clone --branch sarscov2 https://github.com/PaccMann/paccmann_generator
WORKDIR /paccmann_sarsvoc2
# install requirements
COPY .travis/requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt
CMD /bin/bash
