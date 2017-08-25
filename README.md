# Using post-processing and testing features of the pipeline

## Getting Started

Source to get the modified version of DDFacet code

```bash
$ source /net/lofar1/data1/kfchen/software/DDF/init.sh
```

If you have your own version of DDF software, replace:
`$SOFTWARE/DDF/DDFacet/build/lib/DDFacet/Imager/ClassFacetMachine.py` with
`/net/lofar1/data1/kfchen/software/DDF/DDFacet/build/lib/DDFacet/Imager/ClassFacetMachine.py`

### Prepare for Subtaction Pipeline

1. Make sure that all the ms file contains killms_f_ap1 solution. Currently you can change which solution and which template image to use at line 110 in `/net/lofar1/data1/kfchen/software/DDF/ddf-pipeline/scripts/pipeline.py`

2. You can give three parameter in cfg file for the Subtraction code:

```python
### do subtraction when this is not
[image]
fact_reduce_field      = <float> (default 1.0)
                         #Subtracted n times smaller image out
center_ra              = <float> (default None)
                         #The ra of the imaging centre
center_dec             = <float> (default None)
                         #The dec of the imaging centre
```

3. The Sub.py will provide `image_predict_check`, an original size image for checking whether the image is properly subtracted or not.

4. In order to redo prediction, remember to erase `image_predict*`

5. After subtraction but before proceed to further processing, you have to manually stop the pipeline and remove/rename all the sols files related to the original image. (To be improved)

### Prepare for Model Pipeline

1. Create a list of source you want to model. For instance, in `sim.skymodel`:
```python
# (Name, Type, Ra, Dec, I, Q, U, V, ReferenceFrequency="60e6", SpectralIndex="[0.0]", MajorAxis, MinorAxis, Orientation) = format
sim_gauss, GAUSSIAN, 11:13:42.9, +47.30.40.0, 18e-3,,,,125e6, [-0.7], 346.71, 346.71, 62.6
# (Name,Type,Ra,Dec,I, ReferenceFrequency=’55.468e6’, SpectralIndex) = format
sim_delta, POINT, 11:10:39.9, +47.28.40.0, 1,125e6, [-0.56, -0.05212]
```
(See Section 7 in [astron cookbook](https://www.astron.nl/lofarwiki/lib/exe/fetch.php?media=public:lofar_imaging_cookbook_v18.pdf) for further detail on specify model sources)

2. Create a parset file. For instance, in `predict.parset`: (To be automated in the future)
```python
Strategy.InputColumn = DATA_SUB #CAUTION
Strategy.ChunkSize = 300
Strategy.UseSolver = F
Strategy.Steps = [predict4]
Step.predict4.Model.Sources = [sim_gauss, sim_delta]
Step.predict4.Model.Cache.Enable = T
Step.predict4.Model.Gain.Enable = F
Step.predict4.Operation = PREDICT
Step.predict4.Output.Column = SIM_SKYMODEL
Step.predict4.Model.Beam.Enable = True
```

3. Specify your file in the configuration file:
```python
[image]
predict_parset         = <str>   (default None)
                         #Path to parset file for calibrate-stand-alone
sim_skymodel           = <str>   (default None)
                         #Path to skymodel file for simulate observation
```

### Ready to Go?
```bash
python /net/lofar1/data1/kfchen/software/DDF/ddf-pipeline/scripts/pipeline.py tier1.cfg
```

## Remark on the code

### Subtraction Pipelin
Given a image with some area you want to keep and a existing solution, we use the imager to predict and subtract this by the original column.

### Model Pipeline
The idea is to use BBS to generate a column of model sources given a measurement set and add them back to some existing column. Usually you want to keep the pure model column without overwriting it because this step is quite long give a large list of measurement sets. Currently, the model sources are injected in the very begining of the pipeline. You can give it to different stage by changing the location of `add_model` and `add_vis` functions.

### Current Sturcture of the Modified Pipeline

## Prelimary Testing of the Pipeline