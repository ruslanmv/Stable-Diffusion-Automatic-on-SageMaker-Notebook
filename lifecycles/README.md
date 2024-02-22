### Lifecycle configuration to create new instances  environment

For small enviroments you can proceed as follows:

To create a new instance and use the custom environment in that instance, you need to bring the .zip environment from Amazon S3 to the instance. You can do this automatically on the Amazon SageMaker console with the lifecycle configuration script. This script downloads the .zip file from Amazon S3 to the `/SageMaker/` folder on the instance’s EBS unzips the file, recreates the `/envs/` folder, and removes the redundant folders.

1. On the Amazon SageMaker console, under **Admin Configuration,** choose **Lifecycle Configurations (Notebook Instance).**

   ![image-20240217230709350](assets/images/posts/README/image-20240217230709350.png)

2. Select **Create Configuration**

3. Name it `automatic-env`.

On the **Create notebook** tab, enter the following script

> For Start Notebook

```plaintext
## On-Start: After you set up the environment in the instance
## then you can have this life-cycle config to link the custom env with kernel
#!/bin/bash    
sudo -u ec2-user -i <<'EOF'    
ln -s /home/ec2-user/SageMaker/envs/automatic /home/ec2-user/anaconda3/envs/automatic
EOF
echo "Restarting the Jupyter server..."
sudo systemctl restart jupyter-server
```

> For Create Notebook

![image-20240217230841959](assets/images/posts/README/image-20240217230841959.png)

```plaintext
## On-Create: Bringing custom environment from S3 to SageMaker instance
## NOTE: Your SageMaker IAM role should have access to this bucket
#!/bin/bash    
sudo -u ec2-user -i <<'EOF'
aws s3 cp s3://env-automatic/automatic.zip ~/SageMaker/
unzip ~/SageMaker/automatic.zip -d ~/SageMaker/
mv ~/SageMaker/home/ec2-user/SageMaker/envs/ ~/SageMaker/envs
rm -rf ~/SageMaker/home/
rm ~/SageMaker/automatic.zip
EOF
```

![image-20240217230932287](assets/images/posts/README/image-20240217230932287.png)

Now our configuration is ready and let's try to make a new notebook instance with custom configurations and check if the environment is available or not in the newly created notebook instance.

![image-20240217231942888](assets/images/posts/README/image-20240217231942888.png)

![image-20240217232158020](assets/images/posts/README/image-20240217232158020.png)

To import an existing conda environment made in a custom path ("/home/ec2-user/SageMaker/envs/automatic") into the current conda environment, you can use the `conda env create` command. Here are the steps to follow:

1. Activate the current conda environment:
```
source activate python3
```

2. Import the custom conda environment:
```
conda env create --prefix /home/ec2-user/SageMaker/envs/automatic --file /home/ec2-user/SageMaker/envs/automatic/environment.yml
```
Make sure to replace `<current_environment_name>` with the name of your current conda environment.

3. Once the import is complete, you can activate the imported environment:
```
conda activate /home/ec2-user/SageMaker/envs/automatic
```

Now you should be able to work within the imported custom conda environment.

Importing a custom conda environment from SageMaker to your current local conda installation involves a few steps, but can be accomplished using two different methods:



**Method 1: Using `conda env export` and `conda env create`**

1. **On SageMaker:**

   

   - Open a terminal in your SageMaker instance.
   - Activate the environment you want to export: `conda activate automatic` (replace "automatic" with your actual environment name).
   - Export the environment definition: `conda env export > environment.yml`
   - Download the generated `environment.yml` file to your local machine.

2. **On your local machine:**

   - Create a new environment using the exported definition: `conda env create -f environment.yml`
   - Activate the newly created environment: `conda activate automatic` (replace "automatic" with the name you used in the creation step).

**Method 2: Using `conda-pack` and `conda-unpack`**

1. **On SageMaker:**

   

   - Install `conda-pack` in your SageMaker environment: `conda install -c conda-forge conda-pack`
   - Pack the environment: `conda pack -n automatic -o environment.tar.gz` (replace "automatic" with your actual environment name).
   - Download the generated `environment.tar.gz` file to your local machine.

2. **On your local machine:**

   - Install `conda-pack` on your local machine if not already present.
   - Unpack the environment: `conda unpack environment.tar.gz`
   - Activate the newly unpacked environment: `conda activate automatic` (replace "automatic" with the name you used when packing).

**Additional Notes:**

- Both methods achieve the same outcome, but `conda-pack` offers higher portability and compression.
- Ensure you have the same conda channels available locally as on SageMaker for successful package installation.
- Consider package version compatibility between your environments.
- Adjust the environment names accordingly in both methods.

Remember to activate the newly created/unpacked environment to use it for your projects.