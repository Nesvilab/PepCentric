# RESTful API

Users can use the RESTful API to submit queries and retrieve results. The results are in JSON format with straightforward field names.

## API
### Parameters
- `<backend server url>` is `address:port`. The port's default value is 50001.
- `<email>` and `<jobName>` are optional
- `<job type>` can be `peptide` and `protein`
- `<query>` is the peptide or protein (including protein header) sequences
- `<enzyme>` can be `enzymatic` and `nonspecific`
- `<uuid>` is a job ID after submitting the query
- `<protein ID>` is the index of the protein in the JSON retrieved with `<backend server url>/result?jobID=<uuid>`
- `<p-value>` is the p-value threshold
- `<e-value>` is the e-value threshold
- `<q-value>` is the q-value threshold
- `<PSM cap>` maximum number of PSMs retrieved with the `showSpectrum` option
- `<protein ID>` is the index of the protein in the JSON retrieved with `<backend server url>/result?jobID=<uuid>`
- `<sequence ID>` is the index of the sequence in the JSON retrieved with `<backend server url>/showSequence?jobID=<uuid>&proteinID=<protein ID>&q=<q-value>&p=<p-value>&e=<e-value>`
- `<peptide ID>` is the index of the peptide in the JSON retrieved with `<backend server url>/showPeptide?jobID=<uuid>&proteinID=<protein ID>&sequenceID=<sequence ID>&p=<p-value>&e=<e-value>`


### Submit a query
```
<backend server url>/submit?email=<email>&jobName=<job name>&jobType=<job type>&query=<query>&enzyme=<enzyme>
```

### Check the searching status
```
<backend server url>/jobstatus?jobID=<uuid>
```
Retrieved JSON file example
```json
{
    "serveTime": 0,
    "job_complete": "1",
    "status": "ok"
}
```

### Retrieve the search results
Get the highest level (protein level) of the search result
```
<backend server url>/result?jobID=<uuid>
```
Retrieved JSON file example
```json
{
    "searchtime": 3696,
    "servetime": 35,
    "candidates": 193020022,
    "assembletime": 157,
    "indexcount": 5,
    "indexes": "[bioplex, lf, lf02, tmt, tmt16]",
    "digesttime": 119,
    "requests": 1932,
    "sequences": 99,
    "results": [
        {
            "organs": "bile duct;blood;brain;breast;cerebrospinal fluid;colon;head and neck;heart;kidney;liver;lung;multiple;ovary;pancrea;prostate;skin;soft tissue;stomach;uterus",
            "protein_id": 0,
            "dataset_count": 71,
            "protein": "sp|P04637|P53_HUMAN Cellular tumor antigen p53 OS=Homo sapiens OX=9606 GN=TP53 PE=1 SV=p;",
            "diseases": "Alzheimer's disease;Cholangiocarcinoma;Pacreatic cancer, lung cancer, colorectal cancer, breast cancer, stomach cancer, cervical cancer, bladder cancer;Parkinson's disease;SARS-CoV-2;Ulcerative colitis;acute myeloid leukemia;brain cancer;breast cancer;breast cancer, lung cancer, colorectal cancer, leukemia, melanoma, lung cancer, ovary cancer, prostate cancer, kidney cancer;cervical cancer;colon cancer;endometrial carcinoma;head and neck cancer;health;heart failure;hepatocellular carcinoma;kidney cancer;lung cancer;melanoma;multiple cancers;ovarian cancer;ovary cancer;pacreatic cancer;pacreatic cancer, lung cancer, colorectal cancer, breast cancer, stomach cancer, cervical cancer, and bladder cance;prostate cancer;skin cancer;smyotrophic lateral sclerosis;soft tissue cancer;stomach cancer;type 1 diabetes",
            "evalue": "3.567E-25",
            "pvalue": "0.000E+00",
            "best_sequence": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
            "datasets": "BioPlex_293T_3.0;BioPlex_HCT116_1.0;CPTAC2_BCA;CPTAC2_COAD;CPTAC2_COAD_LFQ;CPTAC2_HCC;CPTAC2_OV_JHU;CPTAC2_OV_PNNL;CPTAC3_AML_Joshi;CPTAC3_CCRCC;CPTAC3_C_UCEC;CPTAC3_GBM;CPTAC3_HNSCC;CPTAC3_LSCC;CPTAC3_LUAD;CPTAC3_PDAC;CPTAC3_UCEC;CPTAC3_nCCRCC;IPX0002116000;IPX0003921002;IPX0006050000;MSV000081552;MSV000089012;PDC000219;PDC000248;PDC000315;PDC000356;PDC000358;PDC000360;PDC000362;PDC000408;PDC000433;PDC000434;PDC000514;PTRC_AML;PXD000561;PXD001468;PXD001608;PXD001724;PXD002029;PXD002612;PXD004452;PXD005141;PXD006233;PXD006675;PXD008222;PXD008841;PXD008934;PXD009131;PXD009630;PXD010154;PXD010271;PXD011385;PXD013276;PXD013615;PXD015079;PXD015087;PXD018117;PXD018305;PXD020221;PXD024427;PXD026086;PXD026892;PXD030034;PXD034789;PXD035249;PXD042233;PXD045417;PXD045710;syn25795030;syn26642919",
            "sequences": 97,
            "run_count": 10056
        }
    ],
    "status": "request ok"
}
```

Get the peptide sequence level result
```
<backend server url>/showSequence?jobID=<uuid>&proteinID=<protein ID>&q=<q-value>&p=<p-value>&e=<e-value>
```
Retrieved JSON file example
```json
{
    "searchtime": 3696,
    "servetime": 268,
    "candidates": 193020022,
    "assembletime": 157,
    "indexcount": 5,
    "indexes": "[bioplex, lf, lf02, tmt, tmt16]",
    "protein": "sp|P04637|P53_HUMAN Cellular tumor antigen p53 OS=Homo sapiens OX=9606 GN=TP53 PE=1 SV=p;",
    "digesttime": 119,
    "requests": 1932,
    "sequences": 99,
    "results": [
        {
            "qvalue": "0.000E+00",
            "sequence": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
            "organs": "breast;lung;ovary;soft tissue",
            "dataset_count": 7,
            "sequence_id": 0,
            "diseases": "breast cancer;cervical cancer;lung cancer;ovary cancer;soft tissue cancer",
            "best_psm": "20151123_QEp1_LC7_NiKu_SA_Hek293_Trypsin-bRP01 - 92867",
            "evalue": "3.567E-25",
            "pvalue": "0.000E+00",
            "datasets": "BioPlex_293T_3.0;IPX0006050000;PDC000434;PXD005141;PXD008222;PXD045417;PXD045710",
            "psms": 39,
            "run_count": 18
        },
        {
            "qvalue": "0.000E+00",
            "sequence": "MPEAAPPVAPAPAAPTPAAPAPAPSWPLSSSVPSQK",
            "organs": "breast;colon;kidney;lung;multiple;ovary",
            "dataset_count": 12,
            "sequence_id": 1,
            "diseases": "Pacreatic cancer, lung cancer, colorectal cancer, breast cancer, stomach cancer, cervical cancer, bladder cancer;breast cancer;colon cancer;kidney cancer;lung cancer;multiple cancers;ovary cancer",
            "best_psm": "q5085 - 12637",
            "evalue": "5.082E-20",
            "pvalue": "0.000E+00",
            "datasets": "BioPlex_293T_3.0;CPTAC2_BCA;CPTAC2_COAD;CPTAC3_CCRCC;CPTAC3_LSCC;CPTAC3_LUAD;PDC000408;PXD004452;PXD008222;PXD030034;PXD045417;PXD045710",
            "psms": 70,
            "run_count": 56
        }
    ],
    "status": "request ok"
}
```

Get the peptidoform level result
```
<backend server url>/showPeptide?jobID=<uuid>&proteinID=<protein ID>&sequenceID=<sequence ID>&p=<p-value>&e=<e-value>
```
Retrieved JSON file example
```json
{
    "searchtime": 3696,
    "servetime": 30,
    "assembletime": 157,
    "indexcount": 5,
    "requests": 1932,
    "sequences": 99,
    "sequence": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
    "candidates": 193020022,
    "indexes": "[bioplex, lf, lf02, tmt, tmt16]",
    "protein": "sp|P04637|P53_HUMAN Cellular tumor antigen p53 OS=Homo sapiens OX=9606 GN=TP53 PE=1 SV=p;",
    "digesttime": 119,
    "results": [
        {
            "peptide": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
            "organs": "breast;soft tissue",
            "dataset_count": 4,
            "diseases": "breast cancer;cervical cancer;soft tissue cancer",
            "best_psm": "20151123_QEp1_LC7_NiKu_SA_Hek293_Trypsin-bRP01 - 92867",
            "evalue": "3.567E-25",
            "pvalue": "0.000E+00",
            "datasets": "BioPlex_293T_3.0;IPX0006050000;PXD005141;PXD008222",
            "peptide_id": 0,
            "psms": 12,
            "run_count": 9
        },
        {
            "peptide": "n[229.1629]LLPENNVLSPLPSQAM[15.9949]DDLM[15.9949]LSPDDIEQWFTEDPGPDEAPR",
            "organs": "lung;ovary",
            "dataset_count": 3,
            "diseases": "lung cancer;ovary cancer",
            "best_psm": "Shah_TMT11_QEHFX3_112619_3_11 - 60703",
            "evalue": "3.898E-22",
            "pvalue": "0.000E+00",
            "datasets": "PDC000434;PXD045417;PXD045710",
            "peptide_id": 1,
            "psms": 14,
            "run_count": 9
        }
    ],
    "status": "request ok"
}
```

Get the PSM level result
```
<backend server url>/showSpectrum?jobID=<uuid>&proteinID=<protein ID>&sequenceID=<sequence ID>&peptideID=<peptide ID>&psmCap=<PSM cap>&p=<p-value>&e=<e-value>
```
Retrieved JSON file example
```json
{
    "searchtime": 3696,
    "servetime": 102,
    "peptide": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
    "assembletime": 157,
    "indexcount": 5,
    "requests": 1932,
    "sequences": 99,
    "sequence": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
    "candidates": 193020022,
    "indexes": "[bioplex, lf, lf02, tmt, tmt16]",
    "protein": "sp|P04637|P53_HUMAN Cellular tumor antigen p53 OS=Homo sapiens OX=9606 GN=TP53 PE=1 SV=p;",
    "digesttime": 119,
    "results": [
        {
            "closed_evalue": "46.277 (1.000E-15)",
            "neutral_mass": "4574.117",
            "open_evalue": "46.277 (1.000E-15)",
            "massdiff": "0.986",
            "spectrum": "20151123_QEp1_LC7_NiKu_SA_Hek293_Trypsin-bRP01 - 92867",
            "open_id": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
            "closed_massdiff": "1.994",
            "evalue": "46.278 (3.567E-25)",
            "pvalue": "0.000E+00",
            "closed_id": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
            "open_massdiff": "0.986"
        },
        {
            "closed_evalue": "50.424 (1.000E-15)",
            "neutral_mass": "4575.137",
            "open_evalue": "50.424 (1.000E-15)",
            "massdiff": "2.007",
            "spectrum": "20151123_QEp1_LC7_NiKu_SA_Hek293_Trypsin-bRP08 - 97713",
            "open_id": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
            "closed_massdiff": "2.005",
            "evalue": "50.425 (4.970E-24)",
            "pvalue": "0.000E+00",
            "closed_id": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
            "open_massdiff": "2.006"
        }
    ],
    "status": "request ok"
}
```

Get the number of PSMs given a peptidoform
```
<backend server url>/countSpectrum?jobID=<uuid>&proteinID=<protein ID>&sequenceID=<sequence ID>&peptideID=<peptide ID>&p=<p-value>&e=<e-value>
```
Retrieved JSON file example
```json
{
    "searchtime": 3696,
    "servetime": 0,
    "peptide": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
    "assembletime": 157,
    "indexcount": 5,
    "requests": 1932,
    "sequences": 99,
    "sequence": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
    "candidates": 193020022,
    "indexes": "[bioplex, lf, lf02, tmt, tmt16]",
    "protein": "sp|P04637|P53_HUMAN Cellular tumor antigen p53 OS=Homo sapiens OX=9606 GN=TP53 PE=1 SV=p;",
    "digesttime": 119,
    "results": 12,
    "status": "request ok"
}
```

Dump the search result given a job ID. It might take a while if there are many entries
```
<backend server url>/dump?jobID=<uuid>&q=<q-value>&p=<p-value>&e=<e-value>
```
