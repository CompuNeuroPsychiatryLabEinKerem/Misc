
## ADNI Login
**user** shahar.pery@mail.huji.ac.il
**pswd** adnishaharcnplab

## Parallel Download in Linux

 Concatenate all url-list files
```
>>> echo '\n' >> urlList1.csv
>>> echo '\n' >> urlList2.csv
>>> cat urlList1.csv urlList2.csv > urlListAll.csv
```
Download all URLs (20 stands for the number of URLs)
`>>> cat urlListAll.csv | xargs -n 1 -P 20 wget`
Unzip, *badfiles.txt* will list corrupted files, if any.
`>>> sbatch --wrap="python3 Scripts/UnzipDir.py . DicomsMainDir badfiles.txt"`

## Preparing for BIDS Conversion
<u>**Fact1**</u>: fMRIPrep doesn't cope well with multiple T1s
<u>**Fact2**</u>: fMRIPrep uses a single T1 to all subject's sessions. In the case of a developing disease, that may be wrong.
<u>**Solution**</u>: Use single sessions as "subjects" and in the case of repeated T1 in a single session - take only the last.
 <u>**What to do**</u>: The following code will move the relevant scans to a new folder structure, ready for conversion
```
python3 DivideToSessions.py srcdir dstdir
```
## Converting to BIDS
**Use Heudiconv on Singularity.** On python console do:
```
import os
os.listdir(PREPARED_DCM_DIR)
subjects = sorted(s for s in os.listdir('.') if os.path.isdir(s))
print(' '.join(subjects))
len(subjects)
```
On shell command line do (taking number of subjects (79 here) and subject names from above):
```
>> module load singularity
>> sbatch --array=0-78 --export=DATA_DIR=myproject/data,DCM_DIR=DCM_PREP,BIDS_DIR=BIDS,SUBJECTS="123_S_1300-0 305_S_6899-0 ..." .../Scripts/RunHeudiconv.sh
```
## Checking BIDS Conversion
**Heudiconv sometimes (but rarely) fails!** Unfortunately, for fMRIPrep to run, even on a single subject, ALL the data directory has to be BIDS compatible. The next Python lines run a quick check and move the failed subjects to a different directory.
In Python console do:
```
import re
from glob import glob
getProtocol = lambda funcName: re.search(r'"SeriesDescription": "(.+?)"', open(funcName.split('.')[0]+'.json').read()).group(1)
# This will search the json file and get the protocol name for future stats.
failes = {}

for f in files:
  if len(nb.load(f).shape)==4: continue
  sbj = f.split('/')[-4][4:]
  prot = getProtocol(f)
  failes[sbj] = prot

print(fails)
```  
Having the failed subjects, one can move them to a designated folder.
## Now fMRIPrep, or any other BIDS pipeline, can begin!

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MDQ2Mzg0ODVdfQ==
-->