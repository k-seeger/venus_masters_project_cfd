# -*- coding: utf-8 -*-
"""
Created on Thu Mar 21 12:37:30 2024

@author: Kate
"""


import os
import subprocess
import shutil
import sys

# specify what parameters you want investigated. Only 1 pitch angle per script can be ran, but as many cone angles and omegas as you wish

pitch_angle = -5 
cone_angles = [0, 5, 10]
omegas = [120, 180, 360] # Measured in RPM
vz = 3 

# specify the 'base case' you wish to use
original_case = "copy_case"


# Checks to see that this case exists in the directory that the python script is running from
if os.path.isdir(original_case) is False:
    print("Error: Path ", original_case, " does not exist. Exiting script.")
    sys.exit(0)
else: pass

print("Pitch angle = ",pitch_angle)

# loop through cone angle and omegas
for angle in cone_angles:
    for omega in omegas:       
        print("Cone angle = ", angle)
        ratio = vz/omega
        print("vz/omega ratio =", vz/omega)
        # copy original_case as cone_angle_30 e.g.
        # from https://pynative.com/python-copy-files-and-directories/#h-copy-entire-directory :
        # shutil.copytree(src, dst, symlinks=False, ignore=None, copy_function=copy2, ignore_dangling_symlinks=False, dirs_exist_ok=False
        # define source src and destination dst
        # The dirs_exist_ok dictates whether to raise an exception in case dst or any missing parent directory already exists.
        src = original_case
        dst = "{}_{}_{}".format(angle,omega,pitch_angle)
        
        # check that copied case does NOT already exist (to ensure a simulation isn't overwritten)
        if not os.path.isdir(dst):
            #print("Error: Path ", dst, " already exists. Remove the directory or rename it to run this script.")
            #continue # skips to next angle if this angle exists
        # copy the original case
            try:
                shutil.copytree(src, dst)
                print(f"Directory copied successfully from '{src}' to '{dst}'.")
            except Exception as e:
                print(f"An error occurred: {e}")
                sys.exit(0)
                continue

            print("Making SRFProperties File")
            srf_file_contents = r"""
            /*--------------------------------*- C++ -*----------------------------------*\
              =========                 |
              \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox
               \\    /   O peration     | Website:  https://openfoam.org
                \\  /    A nd           | Version:  10
                 \\/     M anipulation  |
            \*---------------------------------------------------------------------------*/
            FoamFile
            {
                format      ascii;
                class       dictionary;
                location    "constant";
                object      SRFProperties;
            }
            // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
            
            SRFModel        rpm;
            
            origin          (0 0 -5);
            axis            (0 0 -1);
            
            rpmCoeffs
            {
                rpm """            
            srf_file_contents +="{}".format(omega)
            srf_file_contents +=""";
            }
            
            
            // ************************************************************************* //"""

            srf_path = os.path.join(dst,"constant","SRFProperties")
            with open(srf_path, 'w') as file:
                file.write(srf_file_contents)
            # copied from the motorbike tutorial but edited
            #file.close()
            os.chmod(srf_path,0o644)
            
            if os.path.exists(srf_path) is False:
                print("Error: Path ", srf_path, " does not exist. Exiting script.")
                sys.exit(0)
            else: pass
            

            print("Making Allrun file")
            allrun_file_contents = r"""#!/bin/sh
            cd ${0%/*} || exit 1    # Run from this directory
            
            # Source tutorial run functions
            . $WM_PROJECT_DIR/bin/tools/RunFunctions"""
        
            allrun_file_contents +="""
            
            runApplication surfaceTransformPoints "Ry={1}" constant/geometry/original_blade.stl constant/geometry/blade.stl
            mv log.surfaceTransformPoints log.surfaceTransformPoints1
            runApplication surfaceTransformPoints "Ry={0}" constant/geometry/original_blade.stl constant/geometry/blade.stl
            runApplication surfaceFeatures
            runApplication blockMesh
            
            runApplication decomposePar -copyZero
            runParallel snappyHexMesh -overwrite
            runParallel checkMesh
            runParallel SRFSimpleFoam
            #runParallel SRFSimpleFoam -postProcess -func yPlus -latestTime
            runApplication reconstructParMesh -constant
            runApplication reconstructPar -latestTime
            
            runApplication foamLog log.SRFSimpleFoam
            
            #------------------------------------------------------------------------------""".format(angle, pitch_angle)

            allrun_path = os.path.join(dst,"Allrun")
            with open(allrun_path, 'w') as file:
                file.write(allrun_file_contents)

            if os.path.exists(allrun_path) is False:
                print("Error: Path ", allrun_path, " does not exist. Exiting script.")
                sys.exit(0)
            else: pass

		# modify the rights of the file, to give the user run rights
            os.chmod(allrun_path,0o755)

            print("successfully made allrun file")

		# Run the allrun bash script
            Allrun = dst+f"/Allrun"# > {dst}/log_allrun &"
            subprocess.call(Allrun, shell=True)
            print(f"Ran Allrun for a cone angle ({angle}) degrees, omega ({omega}), pitch angle ({pitch_angle}), velocity/omega ratio ({ratio}). Check log for more details")

print("Completed script for pitch angle = ", pitch_angle)
