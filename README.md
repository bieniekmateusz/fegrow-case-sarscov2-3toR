## rgroup-case-sarscov2-3toR
RBFE input files for the reproduction of the study case sarscov2-3toR in rgroup package. 

## BioSimSpace protocols and scripts used to generate the input files for FEP calculations. 

The following details the procedure for protein-ligand binding free energy calculations. 

## File Setup for Free Energy Calculations

Install BioSimSpace (https://github.com/michellab/BioSimSpace/)

### 1) Protein setup: 

  *For use with **AMBER**:*
  - Given the protein in pdb format, we are going to use the [parameterise.py](https://github.com/michellab/BioSimSpace/blob/devel/nodes/playground/parameterise.py) script of BioSimSpace to parameterise it with the ff14SB amber forcefield.
  - Produce AMBER parameterised .rst7 and .prm7 files for the protein: ```parameterise.py --input FILE.pdb --forcefield ff14SB --output FILE```
  
  *For use with **QUBE**:*
  - Given the protein in xml and pdb format, we are going to use the [qube_to_prmRst.py](https://github.com/cole-group/QUBE-SOMD-paper/blob/master/qube_to_prmRst.py) to read the xml/pdb files and generate the corresponding amber files for each fragment:
  - convert the xml file to the AMBER format .rst7 and .prm7: ```qube_to_prmRst.py -x fragX.xml -p fragX.pdb```  
  
### 2) Ligand setup:

  *For use with **AMBER**:*

  - Make a "Ligands" folder in which you will have the pdb files for each ligand and the parameterise.py script
  - create AMBER parameterised .rst7 and .prm7 files for each ligand with: ```parameterise.py --input FILE.pdb --forcefield gaff2 --output FILE```

  *For use with **QUBE**:*

  - Make a "Ligands" folder into which you will copy the QUBE parameterised pdb and xml files for each ligand
  - produce .rst7 and .prm7 files for each ligand: ```qube_to_prmRst.py -p FILE.pdb -x FILE.xml```

**From here the setup is the same for either force field**

### 3a) Solvate the ligands:

  - create solvated, unbound ligand files (e.g. FILE_sol.prm7): ```solvate.py --input FILE.prm7 FILE.rst7 --output FILE_sol --water tip3p --extent 35```
  - `--extent` is the box size used for our system in Angstroms
  
### 3b) Combine the (unsolvated) ligands and protein:

  - make a "complex" folder for the (unsolvated) ligand and protein prm7 and rst7 files
  - for each ligand, combine it with the protein: ```combine.py --system1 LigFILE.prm7 LigFILE.rst7 --system2 PROTEIN.prm7 PROTEIN.rst7 --output PROT_LigFILE```
  - this will create unsolvated prm7 and rst7 files for the ligand in complex with the protein.
  
### 4) Solvate the complex:

  - within the "Complex" folder
  - create solvated prm7 and rst7 files of the ligand in complex with the protein: ```solvate.py --input PROT_LigFILE.prm7 PROT_LigFILE.rst7 --output Complex_sol --water tip3p --box_dim 90```
  - `--extent` is the box size used for our system in Angstroms

### 5) Generate the files for free energy calculations:

  - make a "Perturbations" folder in which you will need the final prm7 and rst7 files for both environments (e.g. ligands in complex, and ligands unbound in solution)
  - create the perturbation files for your free energy calculations (e.g. for a transition of Lig1 to Lig2 the above command will create .mapping, .mergeat0.pdb. .pert, .prm7 and .rst7 files for this pertubration).  Use this command for both bound and unbound environments: ```prepareFEP.py --input1 FILE1_sol.prm7 FILE1_sol.rst7 --input2 FILE2_sol.prm7 FILE2_sol.rst7 --output FILE1_to_FILE2```
  
### 6) Equilibrate the solvated systems:

  - coth the complexes and ligands need to equilibrated
  - create equilibrated rst7 files for the bound and unbound systems (e.g. Lig_sol_eq.rst7 or Complex_sol_eq.rst7): ```amberequilibration.py --input FILE1_to_FILE2_sol.prm7 FILE1_to_FILE2_sol.rst7 --output FILE1_to_FILE2_soleq```

### 7) Free Energy Calculations

1) Create the folder setup:
  - You will need a main folder from which to run the free energy scripts, ```ligand_lambdarun-comb.sh``` and ```complex_lambdarun-comb.sh```, the "pertlist", and where you will also have:
    - A "Perturbations" folder in which will be the pertubations you wish to run, followed by bound and unbound simulation folders containing the relevant files (*i.e.* Lig1_to_Lig2/bound/L1_to_L2_bound.* ) 
    - A "Parameters" folder where you will have the lambda.cfg file

2) The "pertlist"
  - This list details the perturbations you wish to simulate and should include only numbers (*e.g.* 1-2, 2-1 not, Lig1-Lig2 etc.)

3) The lambda.cfg file
  - Configuration files for both our AMBER and QUBE runs can be found in the Parameters/ folder. 
  - There are various parameters which can be altered in these files, namely the number of moves and cycles, the timestep, the type of constraints, the lambda windows used and the platform on which to run the calculation. 

4) The ```ligand_lambdarun-comb.sh``` and ```complex_lambdarun-comb.sh``` scripts
- Script ```ligand_lambdarun-comb.sh``` runs the command for the unbound perturbations, whilst ```complex_lambdarun-comb.sh``` runs the bound perturbations. 
- You will need to specify in both scripts where to read the "somd-freenrg" file from, but if you have used the file structure suggested you should not need to change anything else in these scripts.

5) Once the above is ready, you can start you free energy calculations by running: ```./ligand_lambdarun-comb.sh``` and ```./complex_lambdarun-comb.sh```


## Results

In addition to the above instructions for generating the input files for this paper, the Results/ directory also contains the raw data (free energy calculations and single point energy validations of our SOMD implementation.


