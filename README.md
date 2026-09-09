**Source of truth** : https://git.gregoiremureau.com/openclassroom-ai/oc-p11-bigdata-fruits

```bash
git clone https://git.gregoiremureau.com/openclassroom-ai/oc-p11-bigdata-fruits.git
git clone https://git.gregoiremureau.com/openclassroom-ai/oc-p11-bigdata-fruits.git
```

Dataset: see `fetch-data.sh` (excluded from git if large).

# Big Data Cloud Processing Project - Fruits!

📘 This project is also available in [French 🇫🇷](./README.fr.md)

## 📋 Project Context

This project was carried out as part of a consulting assignment for **Fruits!**, a young AgriTech start-up developing innovative solutions for fruit harvesting. The company aims to preserve fruit biodiversity by enabling species-specific processing through the development of intelligent harvesting robots.

## 🎯 Objectives

Complete a feature extraction processing pipeline in a Big Data environment on the AWS cloud:

1. **Resuming the work** : Take over the incomplete notebook left by a previous apprentice
2. **Completing the pipeline** : Implement the missing parts (weight broadcasting, distributed PCA)
3. **Production deployment** : Deploy the solution on a GDPR-compliant AWS EMR cluster
4. **Cost optimization** : Keep execution costs under €10

The long-term objective is to build a fruit image classification engine for a mobile app raising public awareness.

## 📊 Data

The dataset used is **Fruits-360**, available on Kaggle:
- **147,691 images** of fruits of different varieties
- Format: 100x100 pixels, color
- Specific test set: 103 images of multiple fruits
- **⚠️ Important note** : The dataset is not included in this repository due to its size
- **URL** : https://www.kaggle.com/moltean/fruits

## 📁 Project Structure

```
├── P11_01_notebook.ipynb
├── P11_02_images/
│   ├── Results_PCA/                    # PCA results in Parquet format
│   │   ├── part-00000-*.parquet
│   │   ├── part-00001-*.parquet
│   │   └── ...
│   └── test-multiple-fruits/           # Test images (103 images)
│       ├── apple.jpg
│       ├── apples1.jpg
│       └── ...
└── P11_03_présentation.pdf
```

### 1. `P11_01_notebook.ipynb`

**Main PySpark Big Data processing notebook**

This notebook contains the complete processing pipeline:

- **Initialization** : SparkSession configuration and S3 connection
- **Data loading** : Reading images from S3 in binary format
- **Transfer Learning** : Feature extraction via MobileNetV2 pre-trained on ImageNet
  - Removal of the final classification layer
  - Extraction of 1280-dimensional vectors
  - Broadcasting the model weights to the workers (critical addition)
- **Dimensionality reduction** : Distributed PCA with Spark ML
  - Reduction from 1280 → 70 dimensions
  - 99.94% of variance retained
  - Storage and performance optimization
- **Results export** : Saving in Parquet format on S3

**Key optimizations implemented** :
- Pandas UDF Scalar Iterator for batch processing
- Broadcasting the model weights to avoid repeated loading
- Distributed PCA with parallelized computation on the cluster

### 2. `P11_02_images/`

**Images and results folder**

- **Results_PCA/** : Dimensionality reduction results stored in distributed Parquet format (20 files)
- **test-multiple-fruits/** : 103 varied test images used to validate the pipeline

### 3. `P11_03_présentation.pdf`

**Results presentation**

This presentation summarizes:
- The context and challenges of the project
- The cloud architecture put in place (AWS EMR, S3, EC2)
- The cluster configuration and Spark optimizations
- The complete algorithmic pipeline
- The technical results obtained
- The improvement and deployment perspectives

## 🛠️ Technologies Used

### Cloud & Infrastructure
- **AWS EMR** : Distributed computing cluster (Spark, Hadoop)
- **AWS S3** : Storage of data and results
- **AWS EC2** : m5.xlarge instances (1 Master + 2 Workers)
- **EU-West-3 Region** (Paris) : GDPR compliance

### Big Data & Processing
- **PySpark** : Distributed computing framework
- **Hadoop** : Distributed file system
- **Spark ML** : Distributed Machine Learning library

### Machine Learning
- **TensorFlow** : Deep learning framework
- **MobileNetV2** : Transfer learning model pre-trained on ImageNet
- **PCA** : Dimensionality reduction algorithm

### Development Tools
- **JupyterHub** : Development environment on the cluster
- **SSH Tunneling** : Secure access to the cluster
- **Python 3** : Programming language

## 📈 Technical Architecture

### EMR Cluster Configuration

**Instances** :
- 1 Master node (Spark driver) - m5.xlarge
- 2 Worker nodes (Spark executors) - m5.xlarge
- Region: eu-west-3 (Paris)

**Installed software** :
- Hadoop 3.2.1
- Spark 3.1.2
- JupyterHub 1.4.1
- TensorFlow 2.4.1

**Bootstrap** : Automatic installation of Python packages
```bash
sudo python3 -m pip install numpy pandas pillow pyarrow fsspec s3fs
```

### Secure Access

**SSH Tunneling with Port Forwarding** :
```bash
ssh -i ./emr-keypair.pem -L 8890:localhost:8890 -L 9443:localhost:9443 hadoop@[IP-EMR]
```
- Port 9443: JupyterHub (development)
- Port 8890: Spark monitoring interface

**Advantages** :
- End-to-end encrypted connection
- No external proxy required
- Precise access control

### Processing Pipeline

1. **Image loading** : Binary format from S3, label extraction from paths
2. **Transfer Learning** : MobileNetV2 → 1280-dimension features
   - Broadcasting weights for efficient distribution
   - Pandas UDF Scalar Iterator for batch processing
3. **Distributed PCA** : Reduction from 1280 → 70 dimensions
   - Distributed computation of the covariance matrix
   - Extraction of principal components
   - Feature transformation
4. **Export** : Saving in Parquet format on S3

## 📝 Methodology

1. **Understanding** : Analysis of the existing notebook and identification of gaps
2. **Local development** : Testing and validation on a reduced sample
3. **Cloud migration** : Deployment on the EMR cluster
4. **Optimization** : Adding weight broadcasting and distributed PCA
5. **Validation** : Testing on 103 images, scalability verification

## 💰 Cost Management

**Objective** : < €10 for validation

**Optimization strategies** :
- Use of a modest cluster (1 Master + 2 Workers)
- m5.xlarge instances (performance/cost trade-off)
- Testing on a reduced sample before scaling
- Shutting down the cluster after use
- Local server for development and testing

**Result** : Validation cost < €5 (less than 1h of cluster time)

## 🎯 Results

### Technical

- ✅ Dimensionality reduction: 1280 → 70 dimensions
- ✅ Variance retained: **99.94%**
- ✅ Compression factor: **x4.1**
- ✅ Complete and functional pipeline
- ✅ Validated scalable architecture

### Compliance

- ✅ **GDPR** : EU-West-3 Region (Paris)
- ✅ **Security** : SSH access, secure tunneling
- ✅ **Costs** : < €10 for validation

### Implemented features

- ✅ MobileNetV2 weight broadcasting (missing in the initial version)
- ✅ Distributed PCA with Spark ML (missing in the initial version)
- ✅ Memory optimization with Pandas UDF Scalar Iterator
- ✅ Results export in Parquet format

## 🚀 Outlook

### Short term

**PCA optimization** :
- 70 components valid for 103 test images
- With the full dataset (~150k images), adjust the number of components k
- Test different values to optimize variance/dimensions

**Final classification** :
- Train a classification model on the PCA features
- Evaluate performance on the test set
- Fine-tuning the number of components

### Medium term

**Production deployment** :
- Integration of the model into the mobile application
- Automated CI/CD pipeline
- Real-time performance monitoring
- Model version management

### Long term

**Future extensions** :
- Using the application as a marketing and technical MVP
- Collecting new field data
- Continuous model improvement
- Integration with intelligent harvesting robots

## 🎓 Skills Developed

- Big Data architecture on the cloud (AWS EMR, S3, EC2)
- Distributed processing with PySpark
- Transfer Learning and feature extraction
- Large-scale dimensionality reduction
- Spark performance optimization
- Cloud cost management
- GDPR compliance for data storage

## 👤 Author

**Grégoire Mureau**  
Completion date: October 2025

---

*This project was carried out as part of the AI Engineer program*
