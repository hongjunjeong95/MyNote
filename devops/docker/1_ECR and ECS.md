

# ECR(Elastic Container Repository) & ECS(Elastic Container Service) with GUI



## Whatis ECR?

ECR은 도커 허브 같은 AWS의 컨테이너 저장소다. 먼저 repository를 생성한 후 aws cli를 통해서 docker를 push할 수 있다.



## ECR 만들기 - GUI

1. ECS => Repositories

   <img src="https://user-images.githubusercontent.com/33750210/142514915-25ceb3f1-ea6c-4934-b867-0cd4413aaf9c.png" alt="image" style="zoom:40%;" />

   

2. Click **Create repository**

   ![image](https://user-images.githubusercontent.com/33750210/142515092-b5094df7-4f3c-43c3-bef6-847adf32e6b4.png)

   

3. Set repository name

   <img src="https://user-images.githubusercontent.com/33750210/142515322-db3ee09f-f9b8-491c-8cdf-b29bfb1ae025.png" alt="image" style="zoom:40%;" />

   

4. Click **Create repository**

   <img src="https://user-images.githubusercontent.com/33750210/142515655-93d86b4c-69c3-49ca-b36c-c7f9fe030f63.png" alt="image" style="zoom:50%;" />

## ECR 만들기 2 - CLI

```shell
$ docker login --username AWS -p $(aws ecr get-login-password) <ECR URI>
$ aws ecr create-repository --repository-name <repository name>
```



## Distribute docker image to ECR

```shell
$ docker login --username AWS -p $(aws ecr get-login-password) <ECR URI>
$ docker tag <image_name> <ECR URI>
$ docker push <ECR URI>
```



## Create Cluster

1. ECS => Clusters

   <img src="https://user-images.githubusercontent.com/33750210/142515892-795f1c4d-de42-4fff-89e1-63a7e1c551dc.png" alt="image" style="zoom:40%;" />

   

2. Click **Create Cluster**

   <img src="https://user-images.githubusercontent.com/33750210/142515940-7e5b4172-58e4-453e-9475-0f1520566e93.png" alt="image" style="zoom:40%;" />

   

3. Click **Networking only** and **Next step**

   <img src="https://user-images.githubusercontent.com/33750210/142515998-ef6259cf-0e65-4267-893e-8211ecbd514e.png" alt="image" style="zoom:45%;" />

   <img src="https://user-images.githubusercontent.com/33750210/142516068-ab93ffad-7046-4a14-83d1-cbcbdaccb84c.png" alt="image" style="zoom:50%;" />

   

4. Configure cluster

   <img src="https://user-images.githubusercontent.com/33750210/142517071-d95fef80-b09b-47c7-96da-d28b38586fc9.png" alt="image" style="zoom:40%;" />
   
   <img src="https://user-images.githubusercontent.com/33750210/142517247-65960060-c9cf-4a4f-b0c3-51776fa30f85.png" alt="image" style="zoom:50%;" />
   
   1. Add **Cluster name**
   2. Tag
      * Name : houpang-cluster
   3. **Create**



## Task Definitions

1. ECS => Task Definitions

   <img src="https://user-images.githubusercontent.com/33750210/142517374-a2662067-6e48-403b-b728-f2e4cd2ec220.png" alt="image" style="zoom:45%;" />

   

2. Click **Create Cluster**

   <img src="https://user-images.githubusercontent.com/33750210/142517440-7216d8b0-cea3-4d1d-8ce2-59f5f3062b95.png" alt="image" style="zoom:50%;" />

   

3. Select **FARGATE**

   <img src="https://user-images.githubusercontent.com/33750210/142517488-dfcfa23c-42e5-49aa-8fa8-2fb6a82eec58.png" alt="image" style="zoom:50%;" />

   <img src="https://user-images.githubusercontent.com/33750210/142517572-7c33b596-e302-437c-8051-249c0546614c.png" alt="image" style="zoom:50%;" />

   1. Select **FARGATE**
   2. Click **Next setp**

   

4. Configure task

   <img src="https://user-images.githubusercontent.com/33750210/142517756-da2c68d9-2e43-4651-a5e6-72e610f3e0fc.png" alt="image" style="zoom:50%;" />

   1. Add **Task definition name**

   

5. Set **Task size**

   <img src="https://user-images.githubusercontent.com/33750210/142517884-ee2b230b-a36c-4e39-9d67-2e2619645b60.png" alt="image" style="zoom:40%;" />

   1. Select Task memory
   2. Select Task CPU

   

6. Add container

   ![image](https://user-images.githubusercontent.com/33750210/142518084-4d5fddad-1384-4ad5-9ef2-34393fff63e6.png)

   

7. Container standard options

   <img src="https://user-images.githubusercontent.com/33750210/142518220-908b2de3-5748-495b-ba3a-0ee9c00542e6.png" alt="image" style="zoom:50%;" />

   <img src="https://user-images.githubusercontent.com/33750210/142518387-42a2ecf1-1f72-4725-ac79-6feb644fa217.png" alt="image" style="zoom:50%;" />

   <img src="https://user-images.githubusercontent.com/33750210/142518501-a960aa2b-5904-4f26-8e95-b8afcc35f8e1.png" alt="image" style="zoom:50%;" />

   

   1. Add **Container name**
   2. Add **Image** which is from ECR URI
   3. Set Memory Limits
   4. Set Port for port mappings
   5. Click **Add**

   

8. Click **Create**

   <img src="https://user-images.githubusercontent.com/33750210/142518581-4fff4c94-1f1b-4708-a7e2-ad0cec9435c1.png" alt="image" style="zoom:50%;" />



## Run Task

1. Actions => Run Task

   <img src="https://user-images.githubusercontent.com/33750210/142518718-b4c37546-8963-47f6-99c5-b6db107ad95c.png" alt="image" style="zoom:50%;" />

   

2. Click **Create Cluster**

   <img src="https://user-images.githubusercontent.com/33750210/142519212-6e5f6c30-ba95-4f80-b46e-efa627aed545.png" alt="image" style="zoom:45%;" />

   * Launch type : FARGATE
   * Task Definition
     * Family : houpang-task
     * Revision : 1
   * Platform version : LATEST
   * Cluster : houpang-cluster
   * Number of tasks : 1

   

3. VPC and security groups

   <img src="https://user-images.githubusercontent.com/33750210/142519383-a161314e-3b12-4c30-9ae8-f191872e8905.png" alt="image" style="zoom:45%;" />

   1. Select **VPC**
   2. Select **Subnets**
   2. Edit **SG**
   2. Auto-asign public IP : ENABLED

   

4. Click **Run Task**

   ![image](https://user-images.githubusercontent.com/33750210/142519532-ef9266d7-57c9-498a-bac3-d30beb4dbe95.png)

   

5. Task Status

   <img src="https://user-images.githubusercontent.com/33750210/142519614-7345588a-ccc9-453d-921c-78afebd40eaa.png" alt="image" style="zoom:50%;" />

   * When task status is Running, you can access server by public ip



## 정리

Even though instances were not created and distributed on EC2, you can run the server by ECR and ECS.