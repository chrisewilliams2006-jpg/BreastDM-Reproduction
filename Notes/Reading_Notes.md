Chris Williams RICE UNIVERSTY '29
Dr. Lei Li 
Written 2/26/2026

**PAPER:**

BreastDM: A DCE-MRI Dataset for Breast Tumor Image Segmentation and Classification, Xiaoming Zhao; Yuehui Liao; Jiahao Xie; Xiaxia He; Shiqing Zhang*; Guoyu Wang*; Jiangxiong Fang; Hongsheng Lu; Jun Yu. Computers in Biology and Medicine, Doi:10.1016/j.compbiomed.2023.107255, 2023
(copied and pasted from the link)

**SUMMARY:** 

The paper introduces BREAST-DM, a dynamic contrast-enhanced MRI (DCE-MRI) dataset designed for breast tumor segmentation and benign-versus-malignant classification. According to the paper, there is a major shortage in large public datasets for breast cancer research. The authors created the dataset because there are very few large public DCE-MRI datasets available for developing and comparing deep learning models. BreastDM contains 232 patient cases, including 147 malignant and 85 benign tumors, along with some very helpful annotations by radiologists. To establish baseline performance, the authors evaluated numerous existing segmentation and classification models, including U-Net, DeepLabV3, ResNet, DenseNet, SENet, Vision Transformers, and other state-of-the-art architectures. In addition to these benchmarks, they proposed a new model called the "Local-Global Cross Attention Fusion Network" It is abbreviated as LG-CAFN in the paper. LG-CAFN combines a CNN branch to learn detailed local features with a Vision Transformer branch to capture global contextual information, using a cross-attention mechanism to fuse the two feature types. LG-CAFN is kind of like using two different indepenedent scientists to pool their findings together to get better results, rather than the traditional single model approach. The proposed model achieved the highest classification accuracy and AUC among the methods evaluated on the BreastDM dataset. Overall, the paper gives us a valuable public dataset for breast cancer MRI research that I can use myself for this project.
