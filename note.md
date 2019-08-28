CAF face sketch
测试cycle pix对edge 60的结果 得到较好的一个模型
ok------------------------>
加在服务器上
测试cycle pix对ps2 60的结果 得到较好的一个模型 和图片风格
水墨
油画
数据拷贝脚本
批量测试
研究参数

#  cgan-face-generator
也许是torch版本不一致，模型数据没法用了，函数形参变了，所以需要自己训练吧 还是不对，应该是版本问题
python server.py --dataroot ./datasets/gal  --name caf_pix2pix --model test --which_model_netG unet_256 --which_direction AtoB --dataset_mode single --norm batch
python server.py --dataroot ./data/caf_pix  --name caf_pix --model test --which_model_netG unet_256 --which_direction AtoB --dataset_mode single --norm batch
python server.py --dataroot ./data/caf_pix  --name caf_pix --model test --which_direction AtoB --dataset_mode single

# pytorch-CycleGAN-and-pix2pix
##example
python test.py --dataroot datasets/monet2photo/testA --name monet2photo_pretrained --model test --no_dropout

### data
CycleGAN:
trainA trainB
testA testB

pix2pix: combine_A_and_B
train val 
test
一对一关系 A => B
datasets/
python combine_A_and_B.py --fold_A caf\\edges --fold_B caf\\face --fold_AB caf\\train
python combine_A_and_B.py --fold_A caf\\sketch1 --fold_B caf\\face --fold_AB cafsketch1_pix\\train
python combine_A_and_B.py --fold_A caf\\sketch2 --fold_B caf\\face --fold_AB cafsketch2_pix\\train

### train
python train.py --dataroot ./datasets/caf_cycle --name caf_cycle --model cycle_gan
python train.py --dataroot ./datasets/caf_pix --name caf_pix --model pix2pix --direction AtoB


python train.py --dataroot ./datasets/cafsketch1_cycle --name cafsketch1_cycle --model cycle_gan
*60files 200iters*67s 3.39hous*
python train.py --dataroot ./datasets/cafsketch1_pix --name cafsketch1_pix --model pix2pix --direction AtoB
*60files 200iters*7s 30min*

### test：
python test.py --dataroot ./datasets/caf_cycle --name caf_cycle --model cycle_gan
python test.py --dataroot ./datasets/caf_pix --name caf_pix --model pix2pix --direction AtoB
python test.py --dataroot ./datasets/cafsketch1_cycle/testB/ --name caf_pix --model test --netG unet_256 --dataset_mode single --norm batch

python test.py --dataroot ./datasets/cafsketch1_cycle --name cafsketch1_cycle --model cycle_gan
python test.py --dataroot ./datasets/cafsketch1_pix --name cafsketch1_pix --model pix2pix --direction AtoB
python test.py --dataroot ./datasets/cafsketch1_cycle/testB/ --name cafsketch1_pix --model test --netG unet_256 -dataset_mode single --norm batch

#bash all
```bash
#!/bin/bash

alldirs="cafsketch2 cafsketch1 caf"
LOG_FILE=log.txt

function runpix(){
    dir="($*)_pix"
    #train
    StartTime=$(date +%s.%N)
    python train.py --dataroot ./datasets/${dir} --name ${dir} --model pix2pix --direction BtoA
    Endtime=$(date +%s.%N)

    start_s=$(echo $StartTime | cut -d '.' -f 1)  
    start_ns=$(echo $StartTime | cut -d '.' -f 2)  
    end_s=$(echo $Endtime | cut -d '.' -f 1)  
    end_ns=$(echo $Endtime | cut -d '.' -f 2)  
    time=$(( ( 10#$end_s - 10#$start_s ) * 1000 + ( 10#$end_ns / 1000000 - 10#$start_ns / 1000000 ) ))  
    printf "pix2pix dir $dir\t $time ms \n" >> $LOG_FILE

    #test
    #python test.py --dataroot ./datasets/${dir} --name ${dir} --model pix2pix --direction AtoB
    python test.py --dataroot ./datasets/($*)_cycle/testB/ --name ${dir} --model test --netG unet_256 --dataset_mode single --norm batch
}


function runcycle(){
    dir="($*)_cycle"
    #train
    StartTime=$(date +%s.%N)
    python train.py --dataroot ./datasets/${dir} --name ${dir} --model cycle_gan
    Endtime=$(date +%s.%N)

    start_s=$(echo $StartTime | cut -d '.' -f 1)  
    start_ns=$(echo $StartTime | cut -d '.' -f 2)  
    end_s=$(echo $Endtime | cut -d '.' -f 1)  
    end_ns=$(echo $Endtime | cut -d '.' -f 2)  
    time=$(( ( 10#$end_s - 10#$start_s ) * 1000 + ( 10#$end_ns / 1000000 - 10#$start_ns / 1000000 ) ))  
    printf "cycle_gan dir $dir\t $time ms \n" >> $LOG_FILE

    #test
    python test.py --dataroot ./datasets/${dir} --name ${dir} --model cycle_gan
    #python test.py --dataroot ./datasets/${dir}/testA --name ${dir} --model test --no_dropout
    #python test.py --dataroot ./datasets/${dir}/testB --name ${dir} --model test --no_dropout
}

function runall(){
    for d in $alldirs ; do
        runpix $d
        runcycle $d
    done
}

```

###
python随机选取10000张图片并复制到另一个文件夹中
https://blog.csdn.net/weixin_40769885/article/details/82869760
python实现从一个文件夹下随机抽取一定数量的图片并移动到另一个文件夹
https://blog.csdn.net/weixin_40769885/article/details/82869760