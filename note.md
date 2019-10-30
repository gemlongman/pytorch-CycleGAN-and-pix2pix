CAF face sketch

cycle sketch2 is better
CycleGAN A=》B，A应该是简单的

------------------------>


# Fire up `visdom` server 
for visualization at 
http://localhost:8097

```
conda activate py35
python -m visdom.server

rm -rf 
checkpoints results gygit/test/caf_ 

```
--------------------------------------------------------------------------

# pytorch-CycleGAN-and-pix2pix
pip install -r requirements.txt 
conda install pytorch torchvision cudatoolkit=9.0 -c pytorch

# CycleGAN

## data
CycleGAN:
trainA trainB
testA testB

## train
python train.py --dataroot ~/gygit/test/caf_cycle/caf_cycle --name caf_cycle --model cycle_gan
*60files 200iters*67s 3.39hous cpu:1400s for 1iter 60files*
python train.py --dataroot ./gygit/test/caf_cycle/cafsketch2_cycle --name cafsketch2_cycle --model cycle_gan  --direction BtoA

## test：
python test.py --dataroot ./gygit/test/caf_cycle/cafsketch1_cycle --name cafsketch1_cycle --model cycle_gan

python test.py --dataroot ./gygit/test/caf_cycle/cafsketch2_cycle --name cafsketch2_cycle --model cycle_gan --direction BtoA

python test.py --dataroot ./gygit/test/caf_/cafsketch1_cycle/testB/ --name cafsketch1_pix --model test --netG unet_256 --dataset_mode single --norm batch

--------------------------------------------------------------------------

#pix2pix

## data
pix2pix: combine_A_and_B
train val 
test
一对一关系 A => B
gygit/test/caf_/
python combine_A_and_B.py --fold_A caf\\edges --fold_B caf\\face --fold_AB caf\\train

## train

python train.py --dataroot ./gygit/test/caf_/caf_pix --name caf_pix --model pix2pix --direction AtoB

*60files 200iters*7s 30min*

## test：

python test.py --dataroot ./gygit/test/caf_/caf_pix --name caf_pix --model pix2pix --direction AtoB

python test.py --dataroot ./gygit/test/caf_/cafsketch2_pix/testB/ --name cafsketch2_pix --model test --netG unet_256 --dataset_mode single --norm batch

python test.py --dataroot ./gygit/test/caf_/cafsketch2_cycle/testB --name cafsketch2_cycle --model test --no_dropout

--------------------------------------------------------------------------
##example
python test.py --dataroot gygit/test/caf_/monet2photo/testA --name monet2photo_pretrained --model test --no_dropout

#bash all
```bash
#!/bin/bash

alldirs="cafsketch2 cafsketch1 caf"
LOG_FILE=log.txt

function runpix(){
    dir="($*)_pix"
    #train
    StartTime=$(date +%s.%N)
    python train.py --dataroot ./gygit/test/caf_/${dir} --name ${dir} --model pix2pix --direction BtoA
    Endtime=$(date +%s.%N)

    start_s=$(echo $StartTime | cut -d '.' -f 1)  
    start_ns=$(echo $StartTime | cut -d '.' -f 2)  
    end_s=$(echo $Endtime | cut -d '.' -f 1)  
    end_ns=$(echo $Endtime | cut -d '.' -f 2)  
    time=$(( ( 10#$end_s - 10#$start_s ) * 1000 + ( 10#$end_ns / 1000000 - 10#$start_ns / 1000000 ) ))  
    printf "pix2pix dir $dir\t $time ms \n" >> $LOG_FILE

    #test
    #python test.py --dataroot ./gygit/test/caf_/${dir} --name ${dir} --model pix2pix --direction AtoB
    python test.py --dataroot ./gygit/test/caf_/($*)_cycle/testB/ --name ${dir} --model test --netG unet_256 --dataset_mode single --norm batch
}


function runcycle(){
    dir="($*)_cycle"
    #train
    StartTime=$(date +%s.%N)
    python train.py --dataroot ./gygit/test/caf_/${dir} --name ${dir} --model cycle_gan
    Endtime=$(date +%s.%N)

    start_s=$(echo $StartTime | cut -d '.' -f 1)  
    start_ns=$(echo $StartTime | cut -d '.' -f 2)  
    end_s=$(echo $Endtime | cut -d '.' -f 1)  
    end_ns=$(echo $Endtime | cut -d '.' -f 2)  
    time=$(( ( 10#$end_s - 10#$start_s ) * 1000 + ( 10#$end_ns / 1000000 - 10#$start_ns / 1000000 ) ))  
    printf "cycle_gan dir $dir\t $time ms \n" >> $LOG_FILE

    #test
    python test.py --dataroot ./gygit/test/caf_/${dir} --name ${dir} --model cycle_gan
    #python test.py --dataroot ./gygit/test/caf_/${dir}/testA --name ${dir} --model test --no_dropout
    #python test.py --dataroot ./gygit/test/caf_/${dir}/testB --name ${dir} --model test --no_dropout
}

function runall(){
    for d in $alldirs ; do
        runpix $d
        runcycle $d
    done
}

