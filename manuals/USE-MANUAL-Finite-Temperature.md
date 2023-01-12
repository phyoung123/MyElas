## Calculate thermoelastic constants by MD

### NPT MD

Single NPT MD

```shell
#!/bin/bash

#Generate INCAR
Myelas --vasp incar --encut 500.0 --ismear 1 --sigma 0.1 --ediff 1e-5 --ml 1 --mistart 2 --nsw 20000 --potim 1 -T 300 -ctype npt
#Note: --ml and --mistart are parameters when you use Machine lerning function of VASP>6.3.0

cp INCAR_NPT INCAR
cp POSCAR-uc POSCAR #Supercell
#if not supercell
#phonopy -d --dim="4 4 4" -c POSCAR-uc
#cp SPOCAR POSCAR

# Need KPOINTS and POTCAR

echo "Start Run VASP"
time mpirun -np 32 vasp_std >vasp.log
Myelas -md npt -method 1 -T 300.0 --sstep 5000 --slice 5000 #highly recommended
#Myelas -md npt -method 1 -T 300.0 --sstep 5000 --slice 20000 #Method 2 requires a long time step to ensure convergence
```

Multiple NPT MDs.

```shell
#!/bin/bash

#Generate INCAR
Myelas --vasp incar --encut 500.0 --ismear 1 --sigma 0.1 --ediff 1e-5 --ml 1 --mistart 2 --nsw 20000 --potim 1 -T 300 -ctype npt
#Note: --ml and --mistart are parameters when you use Machine lerning function of VASP>6.3.0

for i in 01 02 03 04 05 #The number must be in the format '01 02 ... 11 12'.
do
# need KPOINTS POTCAR POSCAR-uc and POSCAR_01 ... POSCAR_05
# POSCAR_01 ... POSCAR_05: They are structures generated by random displacements of the same initial structure.
cp INCAR_NPT INCAR
cp POSCAR_${i} POSCAR # Supercell

echo "Start Run VASP"
time mpirun -np 32 vasp_std >vasp.log
cp XDATCAR XDATCAR_${i}
grep 'Total+kin' OUTCAR |awk '{print $2,$3,$4,$5,$6,$7}' >stress_${i}.out
# You can also "cp OUTCAR OUTCAR_${i}" and don't grep stress.out
done

Myelas -md npt -method 1 -T 300.0 --sstep 5000 --estep 20000 -xum 5 #highly recommended
#Myelas -md npt -method 1 -T 300.0 --sstep 5000 --estep 5000  -xum 5 #Method 2 requires a long time step to ensure convergence
```



### NVT MD

```shell
#!/bin/bash

#Generate INCAR
Myelas --vasp incar --encut 500 --ediff 1e-5 --ml 1 --mistart 2 --nsw 20000 --potim 1 -T 300 -ctype nvt
#Note: --ml and --mistart are parameters when you use Machine lerning function of VASP>6.3.0

#Generate strain poscar
Myelas -md nvt -g nvt -smax 0.04 -snum 5
#

root_path=$(pwd)
for i in nelastic_*
do
cd ${i}
for j in strain_*
do
cd ${j}

cp ../../INCAR_NVT INCAR
cp ../../KPOINTS .
cp ../../POTCAR .
cp SPOSCAR POSCAR

#
echo "${i} ${j}"
time mpirun -np 32 vasp_std >vasp.log
grep 'Total+kin' OUTCAR |awk '{print $2,$3,$4,$5,$6,$7}' >stress.out
cd ..
done 
cd ..
done

Myelas -md nvt -so nvt -smax 0.04 -snum 5 -T 300 --sstep 8000 --estep 20000 --slice 3000
```
