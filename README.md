# BoxCars Fine-Grained Recognition of Vehicles
This is Keras+Tensorflow re-implementation of our method for fine-grained classification of vehicles decribed in **BoxCars: Improving Vehicle Fine-Grained Recognition using 3D Bounding Boxes in Traffic Surveillance** ([link](https://doi.org/10.1109/TITS.2018.2799228)).
The numerical results are slightly different, but similar. This code is for **research only** purposes.
If you use the code, please cite our paper:
```
@ARTICLE{Sochor2018, 
author={J. Sochor and J. Špaňhel and A. Herout}, 
journal={IEEE Transactions on Intelligent Transportation Systems}, 
title={BoxCars: Improving Fine-Grained Recognition of Vehicles Using 3-D Bounding Boxes in Traffic Surveillance}, 
year={2018}, 
volume={PP}, 
number={99}, 
pages={1-12}, 
doi={10.1109/TITS.2018.2799228}, 
ISSN={1524-9050}
}
```

	 这篇文章主要是去识别不同的车辆的，其中有一步操作是要把车的3D框得到，
	 然后把长方体的3D box拆开成二维的平面作为特征去学习。
	 根据文章[1]可知，由3个消失点和物体的边缘可以画出该物体的3D框，
	 因此我们希望求得图像的消失点。消失点也是可以根据相机的内参计算得到。
	 在文章中，作者提出：既然很难根据图像去计算消失点（事实上是可以计算得到的，这是后话），
	 那我们不如假设消失点在无穷远处，然后去推测消失点的方向。作者通过量化方法，
	 把方向 -90^{o}～90^{o} 分成60个直方图，每个直方图占 3^{o} ，
	 然后用深度学习去学习这60个直方图，事实上是个分类问题了。
	 
 
## Installation

* Clone the repository and cd to it.

```bash
git clone https://github.com/JakubSochor/BoxCars.git BoxCars
cd BoxCars
```
* (Optional, but recommended) Create virtual environment for this project - you can use **virtualenvwrapper** or following commands. **IMPORTANT NOTE:** this project is using **Python3**.

```bash
virtuenv -p /usr/bin/python3 boxcars_venv
source boxcars_venv/bin/activate
```

* Install required packages 

```bash
pip3 install -r requirements.txt 
```

* Manually download dataset https://medusa.fit.vutbr.cz/traffic/data/BoxCars116k.zip and unzip it.
* Modify `scripts/config.py` and change `BOXCARS_DATASET_ROOT` to directory where is the unzipped dataset.
* (Optional) Download trained models using `scripts/download_models.py`. To download all models to default location (`./models`) run following command (or use -h for help):

```base
python3 scripts/download_models.py --all
``` 


## Usage
### Fine-tuning of the Models
To fine-tune a model use `scripts/train_eval.py` (use -h for help). Example for ResNet50:
```bash
python3 scripts/train_eval.py --train-net ResNet50 
```
It is also possible to resume training using `--resume` argument for `train_eval.py`.

### Evaluation
The model is evaluated when the training is finished, however it is possible to evaluate saved model by running:
```bash
python3 scripts/train_eval.py --eval path-to-model.h5
```


## Trained models
We provide numerical results of models distributed with this code (use `scripts/download_models.py`). 
The processing time was measured on GTX1080 with CUDNN. The accuracy results are always shown as single image accuracy/whole track accuracy (in percents). 
We have also evaluated the method with estimated 3D bounding boxes (see paper for details) and included the results here. 
The estimated bounding boxes are in `data/estimated_3DBB.pkl`. In order to use the estimated bounding boxes, use `--estimated-3DBB path-to-pkl` argument for `train_eval.py` script.
The models which were trained with the estimated bounding boxes have suffix `_estimated3DBB`.  

Net | Original 3DBBs | Estimated 3DBBs | Image Processing Time
----|---------------:|---------------:|---------------------:
ResNet50 |  84.29/91.61 | 81.78/90.79  | 5.8ms
VGG16 | 84.10/92.09 | 81.43/90.68 | 5.4ms
VGG19 | 83.35/91.23 | 81.93/91.48  | 5.4ms
InceptionV3 | 81.51/89.86 | 79.89/89.92 | 6.1ms


## BoxCars116k dataset
The dataset was created for the paper and it is possible to download it from our [website](https://medusa.fit.vutbr.cz/traffic/data/BoxCars116k.zip)
The dataset contains 116k of images of vehicles with fine-grained labels taken from surveillance cameras under various viewpoints. 
See the paper [**BoxCars: Improving Vehicle Fine-Grained Recognition using 3D Bounding Boxes in Traffic Surveillance**](https://doi.org/10.1109/TITS.2018.2799228) for more statistics and information about dataset acquisition.
The dataset contains tracked vehicles with the same label and multiple images per track. The track is uniquely identified by its id `vehicle_id`, while each image is uniquely identified by `vehicle_id` and `instance_id`. It is possible to use class `BoxCarsDataset` from `lib/boxcars_dataset.py` for working with the dataset; however, for convenience, we describe the structure of the dataset also here. 
The dataset contains several files and folders:
* **images** - dataset images and masks 
* **atlas.pkl** - *BIG* structure with jpeg encoded images, which can be convenient as the whole structure fits the memory and it is possible to get the images on the fly. To load the atlas (or any other pkl file), you can use function `load_cache` from `lib/utils.py`. To decode the image (in RGB channel order), use the following statement.
```python
atlas = load_cache(path_to_atlas_file)
image = cv2.cvtColor(cv2.imdecode(atlas[vehicle_id][instance_id], 1), cv2.COLOR_BGR2RGB)
```

* **dataset.pkl** - contains dictionary with following fields
```
cameras: information about used cameras (vanishing points, principal point)
samples: list of vehicles (index correspons to vehicle id). 
		 The structure contains several fields which should understandable. 
		 It also contains field instances with list of of dictionaries 
		 with information about images of the vehicle track. 
		 The flag to_camera defines whether the vehicle is going towards camera or not. 
```

* **classification_splits.pkl** - different splits (*hard*, *medium* from paper and additional *body* and *make* split). Each split contains structure `types_mapping` definig mapping from textual labels to integer labels. It also contains fields `train`, `test`, and `validation` which are lists and each element contains tuple `(vehicle_id, class_id)`.

* **verification_splits.pkl** - similar to classification splits; however, the elements in `train`, `test` are triplets `(vehicle_id1, vehicle_id2, class_id)`.

* **json_data** and **matlab_data** - converted pkl file


## Links 
* [BoxCars116k dataset](https://medusa.fit.vutbr.cz/traffic/data/BoxCars116k.zip)
* Web with our [Traffic Research](https://medusa.fit.vutbr.cz/traffic/)
