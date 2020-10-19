# Object-detection-using-AWS-SageMaker-and-DeepLens
This tutorial will walk you through how to train a custom object detection model and deploy it to DeepLens

First off, you can follow the Custom Object Detection using SageMaker.ipynb. 
It will guide you how to train a custom object detection model, which in the end generate a patched_model.tar.gz file.
This tar.gz file is what we need to create a DeepLens model.

A successful DeepLens project requires a model, which we have trained, and a lambda function.

Luckily, we don't have to write all the codes for the lambda function. 
Instead, we could just download a built-in function of the built-in object detection project, and then modify a few lines of it.

Keeping the majority of the lambda function unchanged, here is what you need to modify.



The output map need to adjust to below classes.

    output_map = {0:'fish'}

The "deploy_ssd_resnet50_300" is the one we converted to. The image sharpe is 300, the frame height and width also need to be 300.

    # The height and width of the training set images
    input_height = 300
    input_width = 300

    # optimize the model
    client.publish(topic=iot_topic, payload='Optimizing model...')
    ret, model_path = mo.optimize('deploy_ssd_resnet50_300', input_width, input_height)
    #model_path = '/opt/awscam/artifacts/mxnet_deploy_ssd_resnet50_300_FP16_FUSED.xml'

    # Load the model onto the GPU.
    client.publish(topic=iot_topic, payload='Loading object detection model')
    model = awscam.Model(model_path, {'GPU': 1})
    client.publish(topic=iot_topic, payload='Object detection model loaded')

    # Set the threshold for detection
    detection_threshold = 0.60

You can directly download the folder deeplens-fish-detection-lambda-function for this fish-detection lambda function.
Just keep in mind that if you want to detect other object, you need to pay attention to the changes of code I made above.


Meanwhile, you need to change some of the function configuration as well.
Go to Basic settings, make sure that the Runtime is Python2.7, the handler is greengrassHelloWorld.infinite_infer_run.
And you had better increase the Memory(MB), which I set it to 1280.



