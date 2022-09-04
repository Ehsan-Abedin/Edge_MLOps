# MLOps

MLOps is an emerging field of ML research that aims to enable and automate ML models into production. According to [sig-mlops](https://github.com/cdfoundation/sig-mlops) MLOps is defined as:

> the extension of the DevOps methodology to include Machine Learning and Data Science assets as first class citizens within the DevOps ecology

In this repository we won't discuss benfites and limitations of MLOps, but we provide some references for those who are interested in using MLOps.

- [Very detailed tutorial in MLOps](https://ml-ops.org/)
- [The difference between DevOps and MLOps](https://hackernoon.com/why-is-devops-for-machine-learning-so-different-384z32f1)
- [AutoML organization and tools](https://www.automl.org/)

**Note:** AutoML is a technology that targets non-expert ML practitioners to build and deploy ML models. It can be used in conjuction with MLOps. However, it is fairly in early stages and we're not going to discuss it here.

## Pipeline

Based on our research and the requirements of the project, we decided to use the following pipeline:

1. **Model and dataset versioning:** As ML-base software is fundamentally different from traditional software, model and dataset versioning is an issue and cannot be handled just by using git (as the amount of data is too large).
2. **Automatic model training:** We will autmoate the training of a face detection/recognition model.
3. **Automatic build:** The process of packaging will be automated (creation of Docker images, building of Docker containers, etc.).
4. **Automatic deployment:** The Docker images will be deployed to a local server automatically.
5. **Model monitoring:** We will provide simple logging and monitoring tools to monitor the performance of the model.
6. **Metadata gathering:** During whole pipeline execution, some metadata will be gathered and stored in the database.
7. **Triggering mechanisms:** The pipeline execution triggering mechanism will be based on pipeline change and manual triggering.

A discussion of currently available tools for each stage of the pipeline is provided below.

### Model and dataset versioning

As mentioned briefly above, ML-base software is different from traditional software in that it is not enough to only have code, but also one need whole dataset to produce the exact model. Plus, the explicit relationship between input and output is not known. So, it requires special attention to versioning.

Git is widely used in versioning and source control for traditional software. However, it is not suitable for ML-base software. The dataset is too large and it is not feasible to index it in git. Models are binary and switching between different versions of the model is not easy. There are other reasons that git alone is not suitable for ML-base software. You can refer to [this](https://github.com/cdfoundation/sig-mlops/blob/main/roadmap/2022/MLOpsRoadmap2022.md#challenges) for more information.

Tools for versioning ML-base software:

1. **[DVC:](https://dvc.org/)** An open-source git-based version control system for ML projects. It is by far the most popular version control system in the wild.
2. **[dotmesh:](https://dotmesh.com/)** According to [dotmesh](https://github.com/dotmesh-io/dotmesh)
   > dotmesh (dm) is like git for your data volumes (databases, files etc) in Docker and Kubernetes

`dotmesh` doesn't have an active community and the latest release was in 2020. So, `DVC` is the best and pretty the only solution for version control. Some important features of `DVC` are (according to [DVC features](https://dvc.org/features)):

- Git-compatible
- Storage agnostic
- Reproducible
- Low friction branching
- Metric tracking
- ML pipeline framework
- Language and framework agnostic
- Track failures

So, we are considering to use DVC as a version and CI/CD app.

Tools for CI/CD

The other option for us is jenkins. This open-source software can provide us with a pipeline that could be run right after the version-control app but it does not provide any version-control itself unlike Gitlab and DVC.

Other option is docker compose but we have the same problem as jenkins. It does not provide any version-control for us.

We are considering the choice between DVC and gitlab as both of these tools are very useful in our case .

### Our choice of tools
We are focusing on developing MLOps techniques for edge devices. Edge devices are quite versatile and designed by different manufacturers. So, we need to have a tool that is compatible with different edge devices and to be device-agnostic. Another important feature is to be framework-agnostic. In other words, we should be able to use all models that are trained with different frameworks, without any modification. To tackle this two issues, we used the following pipeline:

![pipeline](./images/pipeline.png)

`ONNX` standard helps us to be framework agnostic. Almost all training frameworks support ONNX and one can convert the final model to a `.onnx` format and later use it in inference frameworks that support this format (such as `OpenVINO` and `ONNXRuntime`). For inference side, we are going to use `ONNXRuntime`. It is a cross-platform inference engine that supports multiple frameworks and hardware accelerators. So, it's a great choice for edge devices. 

Plus we are using `Docker` to package our application and it's dependencies. It also helps us to create a CI/CD pipeline which is essential for MLOps. As `Docker` it self could be inefficient for edge devices, we are using `balenaOS` which is a lightweight OS, tailored for each hardware with capabilities to run `Docker` containers. Under the hood, `balenaOS` uses `yocto` to build the image file. As of writing this doc, it supports more than 80 devices. More information about `balenaOS` can be found [here](https://www.balena.io/os/).
