# RESTful API

Users can use the RESTful API to submit queries and retrieve results. Every endpoint except `dump` returns JSON with straightforward field names.

There are two ways to access the API:
- **Via the frontend server** (recommended): The frontend server acts as a reverse proxy, forwarding requests to the backend. This is the recommended approach because the backend server is not publicly accessible.
- **Via the backend server**: Connect directly to the backend server if you have network access to it.

Both methods expose the same endpoints and return the same responses.

## Common Parameters
- `<email>` and `<jobName>` are optional
- `<job type>` can be `peptide` and `protein`
- `<query>` is the peptide or protein (including protein header) sequences
- `<enzyme>` can be `enzymatic` and `nonspecific`
- `<uuid>` is a job ID after submitting the query
- `<protein ID>` is the index of the protein in the result JSON
- `<p-value>` is the p-value threshold
- `<e-value>` is the e-value threshold
- `<q-value>` is the q-value threshold
- `<PSM cap>` maximum number of PSMs retrieved with the `showSpectrum` option
- `<sequence ID>` is the index of the sequence in the showSequence JSON
- `<peptide ID>` is the index of the peptide in the showPeptide JSON

Thresholds are optional. Omitted, they do no filtering, except on `dump`, where they default to 0.01.

---

## Option 1: Via the Frontend Server (Recommended)

The frontend server provides a reverse proxy at `/api/<endpoint>` that forwards requests to the backend. Use this when the backend server is not directly accessible.

- `<frontend server url>` is the public URL of the frontend server (e.g., `https://pepcentric.example.com`)
- All endpoints use the pattern: `<frontend server url>/api/<endpoint>?<params>`
- Supported endpoints: `submit`, `jobstatus`, `result`, `showSequence`, `showPeptide`, `showSpectrum`, `countSpectrum`, `dump`, `stats`

### Submit a query
```
<frontend server url>/api/submit?email=<email>&jobName=<job name>&jobType=<job type>&query=<query>&enzyme=<enzyme>
```

### Check the searching status
```
<frontend server url>/api/jobstatus?jobID=<uuid>
```

### Retrieve the search results
Get the highest level (protein level) of the search result
```
<frontend server url>/api/result?jobID=<uuid>&q=<q-value>&p=<p-value>&e=<e-value>
```

Get the peptide sequence level result
```
<frontend server url>/api/showSequence?jobID=<uuid>&proteinID=<protein ID>&q=<q-value>&p=<p-value>&e=<e-value>
```

Get the peptidoform level result
```
<frontend server url>/api/showPeptide?jobID=<uuid>&proteinID=<protein ID>&sequenceID=<sequence ID>&p=<p-value>&e=<e-value>
```

Get the PSM level result
```
<frontend server url>/api/showSpectrum?jobID=<uuid>&proteinID=<protein ID>&sequenceID=<sequence ID>&peptideID=<peptide ID>&psmCap=<PSM cap>&p=<p-value>&e=<e-value>
```

Get the number of PSMs given a peptidoform
```
<frontend server url>/api/countSpectrum?jobID=<uuid>&proteinID=<protein ID>&sequenceID=<sequence ID>&peptideID=<peptide ID>&p=<p-value>&e=<e-value>
```

Dump the search result given a job ID. It might take a while if there are many entries
```
<frontend server url>/api/dump?jobID=<uuid>&q=<q-value>&p=<p-value>&e=<e-value>
```

### Retrieve the index statistics
Get the dataset, file and scan counts of each deployed index. It takes no parameter
```
<frontend server url>/api/stats
```

### Error Responses
The reverse proxy returns HTTP 404 for an unknown endpoint, and HTTP 502 if it cannot reach the backend:
```json
{"status": "error", "error": "Unknown API endpoint: <endpoint>"}
{"status": "error", "error": "Backend server is unavailable"}
{"status": "error", "error": "Failed to connect to backend server"}
```

---

## Option 2: Via the Backend Server (If you have access to the backend server)

Connect directly to the backend server. This requires network access to the backend.

- `<backend server url>` is `address:port`. The port is the one given to the backend's `--port` flag; the reference deployment uses 50001.

### Submit a query
```
<backend server url>/submit?email=<email>&jobName=<job name>&jobType=<job type>&query=<query>&enzyme=<enzyme>
```

### Check the searching status
```
<backend server url>/jobstatus?jobID=<uuid>
```

### Retrieve the search results
Get the highest level (protein level) of the search result
```
<backend server url>/result?jobID=<uuid>&q=<q-value>&p=<p-value>&e=<e-value>
```

Get the peptide sequence level result
```
<backend server url>/showSequence?jobID=<uuid>&proteinID=<protein ID>&q=<q-value>&p=<p-value>&e=<e-value>
```

Get the peptidoform level result
```
<backend server url>/showPeptide?jobID=<uuid>&proteinID=<protein ID>&sequenceID=<sequence ID>&p=<p-value>&e=<e-value>
```

Get the PSM level result
```
<backend server url>/showSpectrum?jobID=<uuid>&proteinID=<protein ID>&sequenceID=<sequence ID>&peptideID=<peptide ID>&psmCap=<PSM cap>&p=<p-value>&e=<e-value>
```

Get the number of PSMs given a peptidoform
```
<backend server url>/countSpectrum?jobID=<uuid>&proteinID=<protein ID>&sequenceID=<sequence ID>&peptideID=<peptide ID>&p=<p-value>&e=<e-value>
```

Dump the search result given a job ID. It might take a while if there are many entries
```
<backend server url>/dump?jobID=<uuid>&q=<q-value>&p=<p-value>&e=<e-value>
```

### Retrieve the index statistics
Get the dataset, file and scan counts of each deployed index. It takes no parameter
```
<backend server url>/stats
```

---

## Examples of Returned JSON files

The backend answers HTTP 200 even on failure, so the outcome must be read from the `status` field.

### submit
The job ID is inside the `status` sentence rather than in a field of its own.
```json
{
    "status": "Sent out the request with query = MPEAAPPVAPAPAAPTPAAPAPAPSWPLSSSVPSQK, jobType = peptide, enzyme = enzymatic, and UUID = f7ac9466-6b7b-4753-8bf3-34ab148a9a9c. Please use this UUID to fetch the result."
}
```

### jobstatus
```json
{
    "serveTime": 0,
    "job_complete": "1",
    "status": "ok"
}
```

Note that an unknown job ID also returns `job_complete` `"0"`, so it cannot be told apart from a
running job here.

### result (protein level)
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
            "peptide_type": 0,
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

### showSequence (peptide sequence level)
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
            "peptide_type": 0,
            "organs": "breast;lung;ovary;soft tissue",
            "dataset_count": 7,
            "sequence_id": 0,
            "diseases": "breast cancer;cervical cancer;lung cancer;ovary cancer;soft tissue cancer",
            "best_psm": "20151123_QEp1_LC7_NiKu_SA_Hek293_Trypsin-bRP01 - 92867",
            "best_hyperscore": "46.278",
            "evalue": "3.567E-25",
            "pvalue": "0.000E+00",
            "datasets": "BioPlex_293T_3.0;IPX0006050000;PDC000434;PXD005141;PXD008222;PXD045417;PXD045710",
            "psms": 39,
            "run_count": 18
        },
        {
            "qvalue": "0.000E+00",
            "sequence": "MPEAAPPVAPAPAAPTPAAPAPAPSWPLSSSVPSQK",
            "peptide_type": 0,
            "organs": "breast;colon;kidney;lung;multiple;ovary",
            "dataset_count": 12,
            "sequence_id": 1,
            "diseases": "Pacreatic cancer, lung cancer, colorectal cancer, breast cancer, stomach cancer, cervical cancer, bladder cancer;breast cancer;colon cancer;kidney cancer;lung cancer;multiple cancers;ovary cancer",
            "best_psm": "q5085 - 12637",
            "best_hyperscore": "44.612",
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

### showPeptide (peptidoform level)
```json
{
    "searchtime": 3696,
    "servetime": 30,
    "assembletime": 157,
    "indexcount": 5,
    "requests": 1932,
    "sequences": 99,
    "sequence": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
    "peptide_type": 0,
    "candidates": 193020022,
    "indexes": "[bioplex, lf, lf02, tmt, tmt16]",
    "protein": "sp|P04637|P53_HUMAN Cellular tumor antigen p53 OS=Homo sapiens OX=9606 GN=TP53 PE=1 SV=p;",
    "digesttime": 119,
    "results": [
        {
            "peptide": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
            "peptide_type": 0,
            "organs": "breast;soft tissue",
            "dataset_count": 4,
            "diseases": "breast cancer;cervical cancer;soft tissue cancer",
            "best_psm": "20151123_QEp1_LC7_NiKu_SA_Hek293_Trypsin-bRP01 - 92867",
            "best_hyperscore": "46.278",
            "evalue": "3.567E-25",
            "pvalue": "0.000E+00",
            "datasets": "BioPlex_293T_3.0;IPX0006050000;PXD005141;PXD008222",
            "peptide_id": 0,
            "psms": 12,
            "run_count": 9
        },
        {
            "peptide": "n[229.1629]LLPENNVLSPLPSQAM[15.9949]DDLM[15.9949]LSPDDIEQWFTEDPGPDEAPR",
            "peptide_type": 0,
            "organs": "lung;ovary",
            "dataset_count": 3,
            "diseases": "lung cancer;ovary cancer",
            "best_psm": "Shah_TMT11_QEHFX3_112619_3_11 - 60703",
            "best_hyperscore": "48.330",
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

### showSpectrum (PSM level)
```json
{
    "searchtime": 3696,
    "servetime": 102,
    "peptide": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
    "peptide_type": 0,
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
            "open_massdiff": "0.986",
            "caution": "no",
            "peptide_type": 0
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
            "open_massdiff": "2.006",
            "caution": "yes",
            "peptide_type": 0
        }
    ],
    "status": "request ok"
}
```

### countSpectrum (PSM count)
```json
{
    "searchtime": 3696,
    "servetime": 0,
    "peptide": "LLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGPDEAPR",
    "peptide_type": 0,
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

### stats
The counts are computed once at start-up.
```json
{
    "indexStats": [
        {
            "name": "lf",
            "datasetCount": 41,
            "fileCount": 6874,
            "scanCount": 812339107
        },
        {
            "name": "tmt",
            "datasetCount": 30,
            "fileCount": 3182,
            "scanCount": 447926612
        }
    ],
    "serveTime": 0
}
```

### dump
Returns a ZIP holding one tab-separated file, or JSON with a `status` field on failure.

A row is one peptide in one run, not one PSM: its PSMs in that run are collapsed to the best e-value,
and `PSM_count` gives how many there were. The columns, in order:

| # | Column | # | Column | # | Column |
| --- | --- | --- | --- | --- | --- |
| 1 | `peptide_sequence` | 12 | `pvalue` | 23 | `closed_evalue` |
| 2 | `peptide` | 13 | `peptide_evalue` | 24 | `open_peptide` |
| 3 | `dataset` | 14 | `peptide_pvalue` | 25 | `open_massdiff` |
| 4 | `run` | 15 | `sequence_evalue` | 26 | `open_hyperscore` |
| 5 | `scan` | 16 | `sequence_pvalue` | 27 | `open_evalue` |
| 6 | `PSM_count` | 17 | `sequence_qvalue` | 28 | `caution` |
| 7 | `organs` | 18 | `protein_evalue` | 29 | `peptide_type` |
| 8 | `diseases` | 19 | `protein_pvalue` | 30 | `job_id` |
| 9 | `mass_diff` | 20 | `closed_peptide` | 31 | `protein_id` |
| 10 | `hyperscore` | 21 | `closed_massdiff` | 32 | `peptide_id` |
| 11 | `evalue` | 22 | `closed_hyperscore` | 33 | `psm_index` |

A row is written only when the protein, sequence, peptide and PSM all pass the thresholds.

---

## The `peptide_type` field
An integer reported for every entry by `result`, `showSequence`, `showPeptide`, `showSpectrum` and
`countSpectrum`, and written as a column by `dump`. It carries two independent bits: a modification
PepCentric did not apply, and a single amino acid variation (SAAV).

| `peptide_type` | Meaning |
| --- | --- |
| 0 | Neither a modification nor a SAAV |
| 1 | Modification, no SAAV |
| 2 | SAAV, no modification |
| 3 | Both a modification and a SAAV |

The modification bit is set when the peptide carries a mass no `*.pepcentricparams` file accounts for
at that position, i.e. one the user wrote into the query. The SAAV bit is set when the sequence occurs
nowhere in the reference proteome but does occur with exactly one residue substituted; sequences
shorter than 8 residues are not compared, and I and L are compared literally. Without
`pepcentric.fasta` only types 0 and 1 appear, and without `pepcentric.params` only 0 and 2.

| Endpoint | in the envelope | on each row |
| --- | --- | --- |
| `result` | — | the protein |
| `showSequence` | — | the sequence |
| `showPeptide` | the sequence | the peptide |
| `showSpectrum` | the peptide | the peptide |
| `countSpectrum` | the peptide | — |
| `dump` | — | the peptide the PSM row belongs to |

Each level is the union of the level below it. The FDR model does not cover a non-zero
`peptide_type`, so the q-, p- and e-values reported beside one are not accurate.

---

## The `best_hyperscore` field
`showSequence` and `showPeptide` report a `best_hyperscore` on every row: the hyperscore of the PSM
named in `best_psm`, as a string to three decimals. No other endpoint carries it. Every score on the
row therefore describes that one PSM, and being a raw rather than an FDR-derived score, it stays
meaningful whatever the `peptide_type`.

Note that it is not the highest hyperscore among the row's PSMs. An e-value is calibrated against the
spectrum it came from, so another PSM can score higher and still have the worse e-value; read
`showSpectrum` for those.

---

## The `caution` field
`showSpectrum` reports a `caution` on every PSM row, and `dump` writes it as a column; nowhere else.
Its value is the string `"yes"` or `"no"`.

`"yes"` marks a PSM for which a peptide of a different sequence competes with the one reported: the
open search scored it higher with a common modification, or the closed search's best identification
differs from the query at one residue by an isotope error or a deamidation. It is a reason to inspect
the spectrum, not a verdict that the identification is wrong.
