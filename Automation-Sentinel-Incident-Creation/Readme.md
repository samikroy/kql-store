






Please use the below button to deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsamikroy%2Fkql-store%2Fmain%2FAutomation-Sentinel-Incident-Creation%2Fazuredeploy.json)

### Deployment

Once deployed you can configure the connetions as below

![image](https://user-images.githubusercontent.com/20562985/196198346-2dbde9ca-9812-4c49-a0e6-d3ebca777bb3.png)


### Configuration

1. Sentinel Connection

![image](https://user-images.githubusercontent.com/20562985/196175586-0fd33803-6fd3-4429-8af4-945c8a0c8511.png)


2. Office 365 Connection

![image](https://user-images.githubusercontent.com/20562985/196175803-51712fbb-1bb4-4279-9d96-64cc24bcf63f.png)

### Post configuration

Once configured, the logic app will look like this.

![image](https://user-images.githubusercontent.com/20562985/195930261-a883dbc0-37ff-401c-87a6-74d4eba7ffea.png)

### Test

Step 1: Send an email to the configure email

![image](https://user-images.githubusercontent.com/20562985/196176523-21e76ca7-705f-468e-beec-aa75b814f742.png)


Step 2: Incident created in Sentinel 

![image](https://user-images.githubusercontent.com/20562985/196183706-02062a9c-eea2-4fd1-9d57-4bf540456341.png)


Thank you for using the tool.

