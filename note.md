#FaceSwap

##useage

---------------------------- A which will be replaced
conda activate py36
A被B替换
1.拆帧
ffmpeg -i ~/gygit/test/video/Supergirl0501_10s.mp4 -vf fps=25 -qscale:v 2 ~/gygit/test/video/IA/Supergirl0501A%d.jpg

ffmpeg -i ~/gygit/test/video/BatwomanS01E01_8s.mp4 -vf fps=25 -qscale:v 2 ~/gygit/test/video/IB/Batwoman%d.jpg


2.从图片提取人脸 参考 faceswap/USAGE.md 
conda activate py36
python faceswap.py extract -i ~/gygit/test/video/IA -o ~/gygit/test/video/FA
python faceswap.py extract -i ~/gygit/test/video/IB -o ~/gygit/test/video/FB 

3.训练学习 
python faceswap.py train -A ~/gygit/test/video/FA -B ~/gygit/test/video/FB -m ~/gygit/test/video/model -t Original -bs 64  -g 1 --iterations 10000 -p


4.转换
python faceswap.py convert -i ~/gygit/test/video/IA -o ~/gygit/test/video/convert -m ~/gygit/test/video/model

5.合并帧
ffmpeg -r 25 -f image2 -y -i ~/gygit/test/video/convertSuperBat/Supergirl0501A%d.jpg -vcodec libx264 -crf 15 -pix_fmt yuv420p ~/gygit/test/video/result.MP4


## General Tips
Note: this script will make grabbing test data much easier, but it is not perfect. It will (incorrectly) detect multiple faces in some photos and does not recognize if the face is the person whom we want to swap. Therefore: **Always check your training data before you start training.** The training data will influence how good your model will be at swapping.

When extracting faces for training, you are looking to gather around 500 to 5000 faces for each subject you wish to train. These should be of a high quality and contain a wide variety of angles, expressions and lighting conditions. 

You do not want to extract every single frame from a video for training as from frame to frame the faces will be very similar.

If you plan to train with a mask or use the Warp to Landmarks option, then you will need to copy the output `alignments.json` file from your source frames folder into your output faces folder for training. If you have extracted from multiple sources, you can use the alignments tool to merge several `alignments.json` files together.

## notes
extract 不要每一帧都提取，尽量不同的脸部
    At the time of writing the best detector is S3FD and the best aligner is FAN
    Currently no models support above 256px
sorting and remove-faces
    可能需要重新对齐
training：
    AB使用同一个权重的encoder，使用不同的Decoder,loss是生成图和原图的差异，反馈给encoder和decoder。转换时候使用另一个decoder 
    Batch Size is the size of the batch that is fed through the Neural Network at the same time.Higher batch sizes will train faster, but will lead to higher generalization. Lower batch sizes will train slower, but will distinguish differences between faces better.
    bs 太大速度快，丢失细节。bs1024在2000张，10000次迭代是出现了loss激增，默认64，为了细节，建议更小一些
    An epoch is one complete representation of the data fed through the Neural Network
    An iteration is one complete batch processed through the Neural Network. 
    -t {dfaker,dfl-h128,dfl-sae,iae,lightweight,original,realface,unbalanced,villain},
            - original: The original model created by /u/deepfakes.
            - dfaker: 64px in/128px out model from dfaker. Enable 'warp-to-landmarks' for full dfaker method.
            - dfl-h128. 128px in/out model from deepfacelab
            - dfl-sae. Adaptable model from deepfacelab
            - iae: A model that uses intermediate layers to try to get better details     https://github.com/deepfakes/faceswap/pull/251
            - lightweight: A lightweight model for low-end cards. Don't expect great results. Can train as low as 1.6GB with batch size 8.
            - realface: A high detail, dual density model based on DFaker, with customizable in/out resolution.
                The autoencoders are unbalanced so B>A swaps won't work so well. By andenixa et al. Very configurable.
            - unbalanced: 128px in/out model from andenixa. The autoencoders are unbalanced so B>A swaps won't work so well. Very configurable.
            - villain: 128px in/out model from villainguy. Very resource hungry (11GB for batchsize 16). Good for details, but more susceptible to color differences.
        测试
            original
            dfl-h128
            villain

