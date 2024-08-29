## Problem Statement
I want to optimize the cost of one of my GCP projects; for which I need to understand the current usage and compare the current cost with the budget. One of the resources I want to optimize is "SSD persistent disk". They are attached to one VM, meaning the VM and disk have a one-to-one relationship. I want to know the exact cost associated with a disk; so in case something with less business impact consumes more storage I could start an effort to optimize it. Right now the billing dashboard would only show me the aggregated cost of all disks within the project. How do I get the exact cost in the billing dashboard associated with each disk?

Some of the points you need to keep in mind while crafting the solution:

- You can only use gcloud cli only to achieve this.
- You don’t need to worry about configurations on the billing dashboard. As soon as you modify resources; they will start reflecting on the billing dashboard.
- Consider this a one-time effort to avoid introducing complexity in the solution. So no need to introduce any new gcp services other than billing and compute engine while crafting the solution.

## Solution
For achieving the above said use-case, I am thinking to use GCP Billing Dashboards for getting the exact cost incurred by the Disks. However, I know that billing dashboard shows aggregated costs for all resources in a project, and not necessarily the cost of individual disks. But, we can use **labels** to filter out the disks we want to monitor from all the GCP's disk (especially SSD persistent disk).

Below are the steps to create a customized billing based on GCP's SSD persistent disk: 

1. First, you'll need to list all your persistent disks and filter them based on their type. Here's the command:
  `gcloud compute disks list --filter="type='pd-ssd'" --format="table(name,zone)"`
2. Once you have the list of SSD persistent disks, you can use the below command to add the label to each disk. You'll need to run this command for each disk you want to label:
   `gcloud compute disks add-labels DISK_NAME --labels=cost-center=storage-ssd-disk --zone=ZONE` (Replace **DISK_NAME** and **ZONE** for each disk listed in above table)
3. (Optional)If you have many disks to label, you can automate this process with a script.
4. Now, you can use the Billing Dashboard on GCP to filter the billing specifically for the Disk. Open the portal and click on **Reports**: 
   ![image](https://github.com/user-attachments/assets/d63d13c9-e43c-4414-b64d-e133454a308d)

5. Choose the filter:
   a. Select the time range.
   b. Group by: *Service*
   c. Select the project.
   d. Select Service: *Cloud Storage*
   e. Labels: {Key: "cost-center", Value: "storage-ssd-disk"}

   ![image](https://github.com/user-attachments/assets/cab9ab52-6a0d-48a3-b491-0502830d5f2f)

6. As we can see the cost associated with all the SSD Persistent disk, due to one-to-one mapping of disk with VM, we can see which disks costing more.
