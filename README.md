# Malicious Behavior Detection using Windows Audit Logs

This is the github page for the AI-Sec 2015 publication "Malicious Behavior Detection using Windows Audit Logs" by Konstantin Berlin, David Slater, and Joshua Saxe. This work will be presented on __Friday October 16, 2015 at 2015 ACM Workshop on Artificial Intelligence and Security__, which is co-located with CCS.

The free pre-print of the publication can be found at http://arxiv.org/abs/1506.04200.

__If you have any questions or issues, or something is not clear about our data or scripts, please report them, so we can fix them as soon as possible.__

###Synoposis

We investigated the utility of agentless detection of malicious endpoint behavior, using only the standard build-in Windows audit logging facility as our signal. We found that Windows audit logs, while emitting manageable sized data streams on the endpoints, provide enough information to allow robust detection of malicious behavior. Audit logs provide an effective, low-cost alternative to deploying additional expensive agent-based breach detection systems in many government and industrial settings, and can be used to detect, in our tests, 83% percent of malware samples with a 0.1% false positive rate. They can also supplement already existing host signature-based antivirus solutions, like Kaspersky, Symantec, and McAfee, detecting, in our testing environment, 78% of malware missed by those antivirus systems.

## Data and Anonymization 

The anonymized version of the data that we used to compute our results can be found [here](https://www.dropbox.com/s/y6zbdgh3t9rl2cd/learn_final_r1.tar.gz?dl=0). The data has been anonomized in order to protect the privacy of the users from which it was collected.

The goal of the anonymization was not only to protect privacy, but also allow the security community to supplement and reuse our data for their own needs. Therefore, part of our data has been anonymized such that if a new set of audit logs are added created by someone, it can be added to our dataset without duplicating feature names.

The following is the description of the anonymization steps

1. Transform all the entries using the regex transformations.
2. Observe all the paths (including directory and name) and the subpaths for files, process names, and registry entries, for all the sandbox derived Windows audit logs. Put them in a bag of public paths called __P__.
3. Encrypt the file/registry/process names in pieces.
    * Do not encrypt any logs from the sandbox runs.
    * For each audit log entry in the enterprise data, see if all or parts of its path is in __P__. Encrypt each directory/registry using its name, if the full path of the directory/registry is not in __P__, otherwise leave unencrypted. The encrypted name is the text `sha1_` followed by the sha1 of the directory/registry name. For files we leave the extension exposed, and only hash the name. Ex. `[windows]\system32\fake_dir\fake.dll` will be encrypted as `[windows]\system32\sha1_<hash of "fake_dir">\sha1_<hash of "fake">.dll`.
    * For sensitive files types, like documents, slides, text, etc., we salt the filenames before hashing.

### Regex Transformations

The regex transformations that we use to generate our feature labels are located in the [regex file](regex.txt). The regex expressions must be executed in order listed to reproduce our results.

### Data Content

The following is the data that we used for our analysis. The file format is specified in Section [File Formats](#ff).

#### Root directory

The root directory contains the following:

* `inter` - directory containing the raw and intermediate file representation of the audit logs (see below)
* `pace_classification.txt` - label classification scores for the cuckoo box data (<0 means unknown label, 0-1 are virus total scores, >1 means malware due to original source of file)
* `pace_column_labels_anon.txt` - string names of the features
* `pace_row_labels_anon.txt` - list of sha1 of the binary files that were ran through CuckooBox. They are in same order as  `pace_classification.txt`, and their row number directly maps into `pace_feature_matrix.txt` row numbers
* `pace_created_labels.txt` - time from epoch when the file was created based on the compile time stamp. If not detected, file created time stamp
* `pace_feature_matrix.txt` - the feature matrix in the format described on the bottom
* `pace_malware_kaspersky_labels.txt` - Kaspersky labels for the CuckooBox data
* `pace_malware_mcafee_labels.txt` - Mcafee labels for the CuckooBox data
* `pace_malware_symantec_labels.txt` - Symantec labels for the CuckooBox data

These are same type of files as described above but for our enterprise dataset (3 users) and splunk dataset (1 user):
* `pace_enterprise_feature_matrix.txt`
* `pace_enterprise_row_labels_anon.txt`
* `pace_splunk_feature_matrix.txt`
* `pace_splunk_row_labels_anon.txt`

#### The `inter` Directory

The `inter` directory contains the intermediate files that we used to form our n-grams. Their names match the *\_row_labels.txt content.

* `*.log` - the JSON formatted intermediate file format. This was the file that we used to create n-grams, and has abstractions of paths and file deletions. The ignored entries, as given by the JSON entry `ignored`, are removed before forming the n-grams.

##### <a name="ff">File Formats</a>
The matrix format written in text is following:
* First line: `<#number of rows> <#number of columns> <#number of non-zero elements in the matrix>`
* This is followed by a list of non-zero entries: `<row> <column> <value>`

## Build

In order to reproduce the plots from the paper using our MATLAB scripts you will you will need MATLAB 2014 or higher, with Statistics and Machine Learning Toolbox. In addition, you will need the [Glmnet](http://web.stanford.edu/~hastie/glmnet_matlab/) MATLAB package in your MATLAB path.

To load the data from disk into MATLAB, type:

```
[A, y, names, virus_kasp, virus_mcafee, virus_symantec, column_labels, t_created, cuckoo_idx, splunk_idx] = read_data(<location of unzipped data txt files>);
```
To create Figure 1 from the manuscript, type:

`make_fig1(A, y, column_labels, virus_kasp, cuckoo_idx, t_created);`

To create Figure 2 from the manuscript, type:

`make_fig2(A, y, cuckoo_idx, splunk_idx, t_created, virus_kasp);`

To create Figure 3 from the manuscript, type:

`make_fig3(A, y, cuckoo_idx, splunk_idx, virus_kasp, virus_mcafee, virus_symantec, t_created);`

## Copyright and License

Code, documentation, and data copyright 2014-2015 Invincea Labs, LLC. Release is governed by [Apache 2.0](LICENSE.txt)  license.

