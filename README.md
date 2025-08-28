PepCentric is a tool that can search proteins and peptides against repository-scale MS/MS spectra (~2 billion spectra) within seconds. It also controls the global false-discovery rate (FDR) for the search results.


# How to use it
- Our public PepCentric website: [http://peptidecentric.org:8080/](http://peptidecentric.org:8080/)
- Our [RESTful API](api.md)


# Use PepCentric via the website
- Users can submit their peptide or protein sequences using the "input query" box or upload a file on the landing page
- Select the input type (protein or peptide)
- Select the enzyme type 
- Fill in optional user information, including analysis name and email
- Click "submit"
- There will be a job ID displayed on the webpage. Please make a note of the ID in case you wish to revisit the results.

There is also a "get results" button that can be used to retrieve old results based on the job ID.

There is also a "dump" button to export all results given a job ID. It might take a long time if there are many entries.


# How to build your own PepCentric server
Get the frontend and backend servers from the following links:

- Frontend server: https://github.com/Nesvilab/PepCentric-frontend
- Backend server: https://github.com/Nesvilab/PepCentric/releases/tag/latest

Set up the servers by following this [tutorial](build_server.md)


# Publications
<a href="https://doi.org/10.1101/2025.02.24.639867" target="_blank">PepCentric Enables Fast Repository-Scale Proteogenomics Searches</a>
<br>
Fengchao Yu, Andy T. Kong, Yi Hsiao, Alexey I. Nesvizhskii
<br>
bioRxiv, 2025, DOI: 10.1101/2025.02.24.639867
