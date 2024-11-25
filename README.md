# Hello World on AWS with Automated CI/CD

This project demonstrates an example of automated deployment of an HTML file to an AWS EC2 instance using **GitHub Actions** to implement CI/CD. The goal is that any change made to the `index.html` file is automatically deployed to an Nginx server running on an AWS EC2 instance.

## Technologies Used
- **AWS EC2**: Service to host the webpage.
- **Nginx**: Web server to serve the HTML page.
- **GitHub Actions**: Deployment automation with CI/CD.
- **SSH** and **scp**: For secure file transfer and server connection.

## Project Architecture
- **GitHub Repository**: Contains the `index.html` file and the **GitHub Actions** workflow (`deploy.yml`).
- **AWS EC2**: AWS instance where the **Nginx** server is hosted to serve the `index.html` file.
- **GitHub Actions**: Automatically detects changes in the `index.html` file and deploys it to the EC2 instance.

## Project Setup

### Step 1: Create an EC2 Instance on AWS
1. Go to the **AWS EC2** console and create a new instance.
2. Select an **Amazon Linux 2** image and a **t2.micro instance** (eligible for the free tier).
3. Configure **security groups** to allow traffic on **ports 22 (SSH)**, **80 (HTTP)**, and **443 (HTTPS)**.
4. Create a key pair to connect via **SSH** (download the `.pem` file).

### Step 2: Configure Nginx
1. Connect to the instance using **SSH**:
   ```bash
   ssh -i "path/to/your/vockey-new.pem" ec2-user@<public-ip>
   ```
2. Install **Nginx** on the EC2 instance:
   ```bash
   sudo amazon-linux-extras install nginx1 -y
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

### Step 3: Assign an Elastic IP
1. In the **AWS EC2** console, assign an **Elastic IP** to your instance to ensure the IP address remains fixed.
2. Associate the **Elastic IP** with the EC2 instance.

### Step 4: Add SSH Key as a Secret in GitHub
1. Go to your repository on **GitHub**.
2. Navigate to **Settings** > **Secrets and variables** > **Actions**.
3. Create a new **secret** named `AWS_EC2_SSH_KEY` and paste the contents of the **`vockey-new.pem`** file.

### Step 5: Create the Workflow File (`deploy.yml`)
1. In the **GitHub** repository, create a directory named `.github/workflows`.
2. Inside that directory, create a file named `deploy.yml` with the following content:
   ```yaml
   name: Deploy to AWS EC2

   on:
     push:
       branches:
         - main

   jobs:
     deploy:
       runs-on: ubuntu-latest

       steps:
         - name: Checkout repository
           uses: actions/checkout@v2

         - name: Set up SSH
           uses: webfactory/ssh-agent@v0.5.3
           with:
             ssh-private-key: ${{ secrets.AWS_EC2_SSH_KEY }}

         - name: Copy files to EC2
           run: |
             scp -o StrictHostKeyChecking=no -i ~/.ssh/aws_key.pem ./index.html ec2-user@107.21.218.122:/tmp/
             ssh -o StrictHostKeyChecking=no -i ~/.ssh/aws_key.pem ec2-user@107.21.218.122 "sudo mv /tmp/index.html /usr/share/nginx/html/index.html && sudo systemctl restart nginx"
   ```

### Step 6: Make Changes and Automatic Deployment
1. Edit the `index.html` file in your code editor (Visual Studio Code).
2. **Commit** and **push** to the `main` branch of the **GitHub** repository.
3. Verify in the **Actions** tab of **GitHub** that the **workflow** runs and deploys the `index.html` file to the EC2 instance.
4. Open your **Elastic IP** address in the browser to verify the changes (e.g., [http://107.21.218.122](http://107.21.218.122)).

## Summary
This project uses a combination of **AWS EC2**, **Nginx**, and **GitHub Actions** to achieve an automated CI/CD flow. Every time you make a change to the `index.html` file and push it to the repository, it is automatically reflected on the EC2 instance without manual intervention. This simplifies the deployment process and demonstrates a simple yet effective implementation of CI/CD.

## Final Considerations
- **Elastic IP**: Ensures that your instance's IP does not change, avoiding constant modifications to the `deploy.yml` file.
- **Nginx Permissions**: Make sure the copied files have the appropriate permissions to be served by Nginx.
- **Security**: Keep your SSH key secure and avoid exposing it publicly.



