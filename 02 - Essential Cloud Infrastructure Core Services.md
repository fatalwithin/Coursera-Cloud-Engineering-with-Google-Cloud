- [Module Overview](#module-overview)
- [Cloud IAM](#cloud-iam)
  - [Organization](#organization)
  - [Roles](#roles)
  - [Members](#members)
  - [Service Accounts](#service-accounts)
  - [Cloud IAM best practices](#cloud-iam-best-practices)
  - [Lab Review: Cloud IAM](#lab-review-cloud-iam)
- [Cloud Storage](#cloud-storage)
  - [Cloud Storage Features](#cloud-storage-features)
  - [Choosing a storage class](#choosing-a-storage-class)
  - [Lab Review: Cloud Storage](#lab-review-cloud-storage)
- [Cloud SQL](#cloud-sql)
  - [Lab Review: Cloud SQL](#lab-review-cloud-sql)
- [Other Database Services](#other-database-services)
  - [Cloud Spanner](#cloud-spanner)
  - [Cloud Firestore](#cloud-firestore)
  - [Cloud Bigtable](#cloud-bigtable)
  - [Cloud Memorystore](#cloud-memorystore)
- [Resource Management](#resource-management)
  - [Cloud Resource Manager](#cloud-resource-manager)
  - [Quotas](#quotas)
  - [Labels and names](#labels-and-names)
  - [Billing](#billing)
  - [Lab Review: Examining Billing Data with BigQuery](#lab-review-examining-billing-data-with-bigquery)
  - [Google Cloud's Operations Suite](#google-clouds-operations-suite)
- [Monitoring](#monitoring)
  - [Lab Review: Resource Monitoring](#lab-review-resource-monitoring)
- [Logging, Error Reporting, Tracing and Debugging](#logging-error-reporting-tracing-and-debugging)
  - [Logging](#logging)
- [Error Reporting](#error-reporting)
  - [Tracing](#tracing)
  - [Debugging](#debugging)
  - [Lab Review: Error Reporting and Debugging](#lab-review-error-reporting-and-debugging)

## Module Overview

In this module we cover Cloud Identity and Access Management or Cloud IAM.  
Cloud IAM is a sophisticated system built on top of email-like address names, job type roles in granular permissions. If you're familiar with IAM from other implementations look for the differences that Google has implemented to make IAM easier to administer and more secure.  

I will start by introducing Cloud IAM from a high-level perspective. We will then dive into each of the components within Cloud IAM which are organizations, roles, members and service accounts. I will also introduce some best practices to help you apply these concepts in your day-to-day work.  
Finally, you will gain firsthand experience with Cloud IAM through a Lab. Let's get started with an overview of Cloud Identity and Access Management.

## Cloud IAM

So what is identity access management?  
It is a way of identifying who can do what, on which resource. The who can be a person, group, or application. The what refers to specific privileges or actions, and the resource could be any GCP service. For example I could give you the privilege or role of compute viewer. This provides you with read-only access to get enlist Compute Engine resources without being able to read the data stored on them.  
Cloud IAM is composed of different objects as shown on this slide. We are going to cover each of these in this module. To get a better understanding of where these fit in, let's look at the Cloud IAM resource hierarchy.  

Google Cloud Platform resources are organized hierarchically as shown in this tree structure.  
**The organization node is the root node in this hierarchy.**  
**Folders are the children of the organization.**  
**Projects are the children of the folders**, and the **individual resources are the children of projects**.  

Each resource has exactly one parent.  
Cloud IAM allows you to set policies at all of these levels. Where a policy contains a set of roles, and role members. Let me go through each of the levels from top to bottom, because resources inherit policies from their parent.  
The organization resource represents your company. Cloud IAM roles granted at this level are inherited by all resources under the organization.  
The folder resource could represent your department.  
Cloud IAM roles granted at this level are inherited by all resources that the folder contains.  
Projects represent a **trust boundary** within your company. Services within the same project have a default level of trust.  
The Cloud IAM policy hierarchy always follows the same path as the GCP resource hierarchy, which means that if you change the resource hierarchy the policy hierarchy also changes. For example, moving a project into a different organization will update the projects Cloud IAM policy to inherit from the new organizations Cloud IAM policy.  

Also, child policies cannot restrict access granted at the parent level. For example, if I grant you that editor role for department X, and I grant you the viewer role at the Bookshelf project level, you still have the editor role for that project. Therefore, it is a best practice to follow the principle of least privilege. The principle applies to identities, roles, and resources. Always select the smallest scope that's necessary for the task in order to reduce your exposure to risk.

### Organization

Let's learn more about the organization node.  
As I mentioned earlier, the organization resource is the root node in the GCP resource hierarchy. This node has many roles, like the **organization admin**.  
The organization admin provides a user like Bob, with access to administer all resources belonging to his organization, which is useful for auditing.  

There is also a **project creator role**, which allows a user like Alice, to create projects within her organization. I am showing the project creator role here because it can also be applied at the organization level, which would then be inherited by all the projects within the organization. The organization resource is closely associated with a G Suite or Cloud Identity Account. When a user with a G Suite or Cloud Identity Account creates a GCP project an organization resource is automatically provisioned for them. Then Google Cloud communicates its availability to the G Suite or Cloud Identity super admins. These super admin accounts, should be used very carefully because they have a lot of control over your organization and all the resources underneath it.  

The G Suite or Cloud Identity super administrators and the GCP organization admin are key roles during the setup process and for lifecycle control, for the organization resource. The two roles are generally assigned to different users or groups, although this depends on the organization structure and needs. In the context of GCP organization setup, the **G Suite or Cloud Identity super administrator** responsibilities are:  
- assign the organization admin role to some users,
- be a point of contact in case of recovery issues, 
- control the lifecycle of the G Suite or Cloud Identity account and organization resource.  

The responsibilities of the **organization admin** role are:
- define IAM policies,
- determine the structure of the resource hierarchy,
- delegate responsibility over critical components such as networking, billing, and resource hierarchy, through IAM roles.

Following the principle of least privilege, this role does not include the permission to perform other actions, such as creating folders. To get these permissions, an organization admin must assign additional roles to their account. For more information about creating and managing organizations, see the how-to guide, in the link section of this video.  

Let's talk more about folders, because they can be viewed as sub organizations within the organization. Folders provide an additional grouping mechanism and isolation boundary between projects. **Folders can be used to model different legal entities, departments, and teams within a company**. For example, a first-level of folders can be used to represent the main departments in your organization, like departments x and y. Because folders, can contain projects in other folders, each folder could then include other subfolders to represent different teams, like teams A and B. Each team folder could contain additional subfolders, to represent different applications, like products 1 and 2.  

**Folders allow delegation of administration rights**, for example, each head of a department, can be granted full ownership of all GCP resources that belong to their department. Similarly, access to resources can be limited by folder, so users in one department can only access and create GCP resources, within that folder. Let's look at some other resource manager roles, while remembering that policies are inherited from top to bottom.  
The organization node also has a **viewer** role. They grants view access to all resources within an organization.  
The folder node has multiple roles that mimic the organizational roles, but are applied to resources within a folder.  
There is an admin role that provides full control over folders. A **creator** role, to browse the hierarchy and create folders, and a viewer role, to view folders and projects below a resource.  
Similarly for projects, there is a creator role that allows a user to create new projects, making that user automatically the owner. There is also a **project deleter role** that grants deletion privileges for projects.

### Roles

Let's talk more about roles which define the can do what on which resource part of Cloud IAM. There are three types of roles in Cloud IAM, basic roles, predefined roles, and custom roles. Basic roles are the original roles that were available in the Cloud console, but they are broad. You apply them to a Google Crowd project, and they affect all resources in that project.  
In other words, IAM basic roles offer fixed, coarse-grained levels of access. The basic roles are the **owner, editor and viewer** roles.  

The **owner** has full administrative access. This includes the ability to add and remove members and delete projects.  
The **editor** role has modify and delete access. This allows the developer to deploy applications and modify or configure its resources.  
The **view** role has read only access.  

Now all of these roles are concentric. That is the owner role includes the permissions of the editor role. And the editor role includes the permissions of the viewer role.  
There is also a **billing administrator** role to manage billing and add or remove administrators without the right to change the resources in the project. Each project can have multiple owners, editors, viewers and billing administrators. GCP services, offers their own set of predefined roles, and they define where the roles can be applied. This provides members with granular access to specific GCP resources and prevents unwanted access to other resources. These roles are a collection of permissions, because to do any meaningful operations, you usually need more than one permission.  

For example, as shown here, a group of users is granted the instance admin role on project a. This provides the users of that group with all the Compute Engine permissions listed on the right and even more. Grouping these permissions into a role makes them easier to manage. The permissions themselves are classes and methods in the API's. For example, compute instance start can be broken down into the service, resource and verb. That mean that the permission is used to start a stopped Compute Engine instance. These permissions usually line with the actions corresponding REST API.  

Compute Engine has several predefined IAM roles. Let's look at three of those.  
The **Compute Admin** role provides full control of all Compute Engine resources. This includes all permissions that start with compute, which means that every action for any type of Compute Engine resource is permitted.  
The **Network Admin** role Contains permissions to create, modify and delete network resources, except for firewall rules and SSL certificates. In other words, the network admin role allows read only access to firewall rules SSL certificates, and instances to view their ephemeral IP addresses.  
The **storage admin** role contains permissions to create, modify, and delete disks, images, and snapshots. For example, if your company has someone who manages project images, and you don't want them to have the editor role in the project. Grant their account the storage admin role on that project. For the full list of predefined roles for Compute Engine, see the links section in the slides.  

Now, roles are meant to represent abstract functions and are customized to line with real jobs. But what if one of those roles does not have enough permissions? Or you need something even finer grained? That's what custom roles permit. A lot of companies use the least privileged model in which each person in your organization is giving the minimal amount of privilege needed to do their job. Let's say you want to define an instance operator role to allow some users to start and stop Compute Engine virtual machines, but not reconfigure them. Custom roles allow you to do that.

### Members

Let's talk more about members which define the who part of who can do what on which resource.  
There are five different types of members.  
- Google Accounts,
- Service Accounts,
- Google Groups,
- G Suite domains
- Cloud Identity domains.  

A **Google account** represents a **developer, an administrator, or any other person** who interacts with GCP. Any email address that is associated with a Google account can be an identity including gmail.com or other domains.  
New users can sign up for a Google account by going to the Google account sign-up page without receiving mail through Gmail.  

A **service account** is an account that **belongs to your application instead of to an individual end user**. When you run code that is hosted on GCP, you specify the account that the code should run as, you can create as many service accounts as needed to represent the different logical components of your application.  

A **Google group** is a named collection of Google accounts and service accounts. Every group has a unique email address that is associated with the group. **Google Groups are a convenient way to apply an access policy to a collection of users**. You can grant and change access controls for a whole group at once, instead of granting, or changing access controls one at a time for individual users, or service accounts.  

**G Suite domains** represent your organization's internet domain name, such as example.com, and when you add a user to your G Suite domain, a new Google account is created for the user inside this virtual group, such as username@example.com. GCP customers who are not G Suite customers can get the same capabilities through Cloud Identity.  

**Cloud Identity** lets you manage users in groups using the Google Admin console, but you do not pay for or received G Suite's collaboration products such as Gmail, Docs, drive and calendar. Cloud Identity is available and free in premium editions.  
The **Premium edition adds capabilities for mobile device management**. Now it's important to know that you can not use Cloud IAM to create, or manage your users, or groups, instead, you can use Cloud Identity or G Suite to create and manage users. What if you already have a different corporate directory? How can you get your users and groups into GCP?  

Using **Google Cloud directory sync**, your administrators can login and manage GCP resources using the same usernames and passwords they already use. This tool synchronizes users and groups from your existing Active Directory, or LDAP system with the users and groups in your Cloud Identity domain. The synchronization is one-way only, which means that no information in your Active Directory or LDAP map is modified.  
Google Cloud Directory Sync is designed to run scheduled synchronizations without supervision, after it's synchronization rules are set up. GCP also provide single sign-on authentication. If you have your identity system, you can continue using your own system and processes with SSO configure. When user authentication is required, Google will redirect to your system.  
If the users are authenticated in your system, access to Google Cloud Platform is given, otherwise, the user is prompted to sign in. This allows you to also revoke access to GCP. If your existing authentication system support SAML2 SSO configuration is as simple as three links and a certificate as shown on this slide. Otherwise, you can use a third party solution like ADFS, ping, or Okta. For more information about using your existing identity management system, see the link section of this video. Also, if you want to use a Google account but are not interested in receiving mail through Gmail, you can still create an account without Gmail. For more information about this, see the link section of this video.

### Service Accounts

As I mentioned earlier another type of member is a **service account**. A service account is an account that belongs to your application instead of to an individual end-user. This provides an identity for carrying out server-to-server interactions in a project without supplying user credentials.  
For example, if you write an application that interacts with Google Cloud Storage. **It must first authenticate** to either the Google Cloud Storage XML API. Or JSON API. You can **enable service accounts**. And **grant read write access to the account** on the instance where you plan to run your application. Then program the application to obtain credentials from the service account. Your application authenticates seamlessly to the API without embedding any secret keys. Or credentials in your instance, image or application code.  

Service accounts are **identified by an email address** like the example shown here. There are three types of service accounts. User created or custom built in and Google API's service accounts. By default, all projects come with a built in compute engine default service account. Apart from the default service account.  
All projects come with the Google Cloud Platform API's service account. Identifiable by the email project dash number at cloud services.g service account.com. This is a service account designed specifically to run internal Google processes on your behalf. And it is automatically granted the editor role on your project. Alternatively, you can also start an instance with a custom service account.  

Custom service accounts provide more flexibility than the default service account. But they require more management from you.  
You can create as many custom service accounts as you need.  
Assign any arbitrary access scopes. Or cloud Iam roles to them. And assign the service accounts to any virtual machine instance.  
Let's talk more about the **default** Compute Engine service account. As I mentioned, this account is automatically created per project.  
This account is identifiable by the email. Project dash number dash compute at developer.gserviceaccount.com. And it is automatically granted the editor role on the project. When you start a new instance using G Cloud. The default service account is enabled on that instance. You can override this behavior by specifying another service account. Or by disabling service accounts for the instance.  
Now authorization is the process of determining what permissions an authenticated identity has on a set of specified resources.  

**Scopes** are used to determine whether an authenticated identity is authorized. In the example shown here. Applications A and B contain authenticated identities. Or service accounts.  
Let's assume that both applications want to use a Cloud Storage bucket. They each request access from the Google authorization server. And in return they receive an access token.  
Application A receives an access token with read only scope. So it can only read from the Cloud Storage bucket.  
Application B in contrast, receives an access token with readwrite scope. So it can read and modify data in the cloud storage bucket. Scopes can be customized when you create an instance using the default service account. As shown in this screenshot. These scopes can be changed after an instance is created by stopping in.  

Access scopes are actually the legacy method of specifying permissions for your VM. Before the existence of IAM roles access scopes were the only mechanism for granting permissions to service accounts.  
For user created service accounts use **Cloud IAM roles** instead to specify permissions. Now roles for service accounts can also be assigned to groups or users. Let's look at the example shown on this slide.  
First, you create a service account that has the instance admin role. Which has permissions to create, modify and delete virtual machine instances and discs.  
Then you treat this service account as the resource. And decide who can use it by providing users. Or a group with a service account user role. This allows those users to act as that service account to create, modify. And delete virtual machine instances and discs. Users who are service account users for a service account can access all the resources that the service account has access to. Therefore be cautious when granting the service account user role to a user or group.  

Here is another example. The VMs running component one. Are granted editor access to project B using service account one. VMs running component two are granted object viewer access to bucket one. Using an isolated Service account two. This way you can scope permissions for VMs without recreating VMs.  
Essentially Cloud IAM. lets use slice a project in two different microservices each with access to different resources. By creating service accounts to represent each one. You assign the service accounts to the VMs when, they are created. And you don't have to ensure the credentials are being managed correctly. Because GCP manages security for you.  

Now, you might ask. How are the service accounts authenticated? Although users require a username and password to authenticate service accounts, use **keys**. There are two types of service account keys. GCP managed keys and user managed keys. GCP managed keys are used by GCP services such as App Engine and Compute Engine. These keys cannot be downloaded and are automatically rotated. And used for a maximum of two weeks. User managed keys are created, downloadable. And managed by users. When you create a new key pair, you download the private key. Which is not retained by Google. With user managed keys. You are responsible for security of the private key. And other management operations. Such as key rotation which is illustrated on this slide.

### Cloud IAM best practices

Let's talk about some Cloud IAM best practices to help you apply the concepts you just learned in your day-to-day work.  
First, **leverage and understand the resource hierarchy**. Specifically, use projects to group resources that share the same trust boundary.  
Check the policy granted on each resource and make sure you recognize the inheritance. Because of inheritance, use the principle of least privilege when granting roles.  
inally, audit policies using Cloud audit logs and audit memberships of groups using policies.  

Next, I recommend **granting roles to groups instead of individuals**. This allows you to update group membership instead of changing a Cloud IAM policy. If you do this, make sure to audit membership of groups used in policies and control the ownership of the Google group used in Cloud IAM policies.  
You can also use multiple groups to get better control. In the example on this slide, there is a network admin group. Some of those members also need a read write role to a Cloud Storage bucket, but others need the read only role.  
Adding and removing individuals from all three groups controls their total access. Therefore, groups are not only associated with job roles but can exist for the purpose of role assignment.  

Here are some best practices for using service accounts. As mentioned before, **be very careful when granting the service accounts user role** because it provides access to all the resources of the service account has access to. Also when you create a service account give it a display name that clearly identifies its purpose, ideally using an established naming convention. As for keys, **establish key rotation policies and methods and audit keys with the serviceAccount.keys.list** method.  

Finally, I recommend using **Cloud Identity Aware Proxy or Cloud IAP**. Cloud IAP lets you establish a central authorization layer for applications accessed by HTTPS.  
So you can use an application level access control model instead of relying on network level firewalls. Applications and resources protected by Cloud IAP can only be accessed through the proxy by users and groups with the correct Cloud IAM role.  
When you grant a user access to an application or resource by Cloud IAP, they're subject to the fine-grained access controls implemented by the product in use without requiring a VPN.  
Cloud IAP performs authentication and authorization checks when a user tries to access a Cloud IAP secure resource as shown on the right. For more information about Cloud IAP, see the link section of this video.

### Lab Review: Cloud IAM

In this lab, you granted and revoked Cloud IAM roles, first to a user username 2, and then to a service account user. Having access to both users allow you to see the results of the changes you made. You can stay for a lab walk-through, but remember that GCP's user interface can change. So your environment might look slightly different. Welcome to the walk-through of the Cloud IAM lab. In this lab, we have set up two users for you, and at this point I have logged into the console as username 1. So Qwiklabs will provide you with two usernames to log into and we'll do some operations with both, but right now I have already logged in as username 1. So the first instruction tells you to log into the console in another tab as username 2. So console, I'm going to grab that username, certainly going here, I'm going to say add account, username 2, and luckily we've been given the same password for both usernames and login here, and with Qwiklabs you have new accounts. So it's always going to ask you for all of this new user acceptance, and I'll accept this terms, I'm good to go. So task two is to explore the IAM console. So I'm going to go to the username 1 tab. I'm going to go to IAM, and I'm going to click on there. If I hit Add, I can look around at the different roles I can provide. Let me go ahead and click Cancel, feel free to explore as much as you want, you can see here there are roles based on different products and services that we have. I'll hit Cancel. Let me go to username 2, and I'm going to do the same thing. Let me go to IAM.
So I'm going to browse this list now and I am going to look for the names associated with username 1, which in my case ends in 82462, I see it here, and here is username 2. You can see there are different roles associated with each one of them. Username 1 has App Engine admin, BigQuery admin, editor, owner, and viewer. Whereas, username 2, which is the one that I'm logged in in this tab only has Viewer Access.
So now I'm going to move on to task three. So I'm going to go back to username 1, there's going to go to be a lot of switching back and forth in this lab. So make sure you keep track of which tab is username 1 and which is username 2. I am going to go to Google Cloud Storage here, and I'm going to create a bucket in here. So buckets need to be globally unique. So I am going to use my Cloud Project ID because it is pretty unique, and I'm going to click Create and keep all the other defaults, and make sure to note the name of your bucket because we'll use it as your bucket name across the lab. So here I'm going to go to upload files, and let me find just any sample file here just a screenshot, and I've uploaded it there. Once it's uploaded, I'm going to rename it here, and I'm going to call it sample.txt. The reason I'm doing this is because it's going to be much easier to run any of the commands I'm going to do with sample.txt as a name as opposed to that long name I already had. So at this point in the lab, you can hit the Check my progress button inside the lab and it'll give you a green check and five points. If you've correctly created a bucket and uploaded a sample file. So now I'm going to switch to username 2 and I'm going to go to storage browser. I'm going to verify that username 2 has view access to that bucket, and here it is. Because it's inherited that, I can view the sample file. Task four is I'm going to remove project viewer role for username 2. So in order to do that I have to go back to username 1. I'm going to go back to IAM,
and then I'm going to find username 2 which is this one right here, 73. I'm going to hit Edit, and then I am going to hit the garbage can so that I can remove it. I hit Save, and then I at this point I can also check my progress and I'll get five points and a green check mark. I should have 10 points out of 20 in the lab. If I have properly done that. Throughout these labs if you ever get to a point where you realize that you didn't get the points necessary, it's probably because you missed a step or two, granted sometimes the lab is actually broken because they're based on technology that changes a lot. But if a lab isn't broken, chances are you just missed a step. So I usually recommend go back three steps. Check your work, make sure you did everything. Usually, that's what happened. So now we're going to verify that the username 2 has lost access. So I'm going to go back to the username 2 bucket tab, and then I'm going to click Home, and then I'm going to go back to storage to verify. I could've just refresh the screen as well, Refresh. List of buckets could not be loaded. So as you can see I do not have access anymore. So now the next task, task five, is to add Storage Access. So I'm going to copy the value of username 2 from the Qwiklabs lab name, from their connection details on the left of your lab instructions. So I'll copy that, I'm going to go back to username 1 tab, and I'm already in I am. So I'm going to hit Add, and then for new members I'm going to paste the value here. That is it, and I am going to select Storage, scroll down. Luckily it's alphabetical, and I am giving it Storage Object Viewer. Then I'm going to hit Save. This is another checkpoint in the lab where you can go back and hit Check my progress, and you should get another five points that you have checked, that you have actually provided the right permissions. Now we had one in the modules that sometimes the permissions upload faster than will be displayed in the GCP Console. Sometimes you just have to be patient. Maybe click Check my progress, wait a couple seconds if you didn't get it, and then you'll get the five points and the green check. So the next piece of task 5 is to verify that username 2 now has storage access. So if I go back here,
and I'm going to start Cloud Shell. Because username 2 doesn't have project viewer roles, so it won't be able to see anything in the console, but we can see things in Cloud Storage. So we're going to use Cloud Shell for that. So let's make sure I know my bucket name. I copied it earlier, but I have definitely forgotten it by now. So let me go back here to Storage easily, and I can easily copy paste the bucket name here. Copy that, and back in Cloud Shell, I am going to do a gsutil ls to list for that bucket, gs://my bucket name, and username 2 should be able to see that there's sample.txt in the bucket and there it is. So now I can close username 2 tab because the rest of the lab is done in the username 1 console. So task 6 is to set up the service account user. So in IAM, I'm going to go to Service accounts. I'm going to create a service account, and the service account name is going to be read-bucket-objects, and I'm going to hit Create. It's going to ask me which role to provide, and I am going to be giving it Storage, Storage Object Viewer. Hit Continue, and then I'm going to hit Done. So now we've created our service account, so we're going to go back to the main IAM page, and we are going to select the service account we just created, and we're going to hit Add. In order to add members, normally you could perform this activity for a specified user group or a domain. But for training purposes and for this video, we're just going to grant the service account user role to everyone at a company called autostrat.com, which is a fake company used for demonstrating and training. So the new member is going to be autostrat.com, and I am going to give it Service Accounts,
Service Account User, and then I'm going to hit Save. So now I'm going to go back to IAM, and I am going to add,
and I am going to provide compute engine access. So the new member is autostrat.com. Make sure you're typing it correctly. I am giving it Compute Engine,
and Compute Instance Admin V1 and save. So essentially, that step is a rehearsal of activity that you would probably perform for a specific user. It gives the user limited abilities with a VM Instance. It would be able to connect via SSH to a VM and perform possibly some administration tasks. So now, I am going to create a VM with the service account I created.
Create, I am going to use the same name provided in a lab, demoIAM. I'm using us-central1. The zone is us-central1-c, and the machine type is an F1-micro. It is just for demonstration purposes. So let's not waste resources, and the service account is the read bucket objects account, and I'm going to hit Create. So this is another checkpoint in the lab, and you should be able to hit Check my progress, and verify that you have gotten the last five points in the lab. Again, this is another one that might take a couple seconds to propagate. So just give it a second, and make sure that you get the green check in the final five points. Task 7, you explore the service account user role, and now you've already completed all of the tasks in the labs. So this is just for learning purposes. So I'm going to go in here, and I'm going to SSH into this account into the VM that I just created. Then I am going to run a gcloud compute instances list. I am expecting to see an error because I do not have the correct permissions to list those for my project. Just wait for that to show up, and there you can see error. Some request did not succeed because I don't have permission to do that. So now I'm going to try to copy the file from the bucket that I created earlier. So my bucket name is the Project ID which I've forgotten already.
Here gets/sample.txt,
and you can see it successfully copied it. Now I'm going to copy it into another file,
and then I'm going to try to upload into my bucket. Bucket name is here,
and you'll see I can download, but I cannot add. In review in this lab, you granted and revoked Cloud IAM roles, first for user, and then to service account user. I hope you enjoyed the walkthrough. Thank you.

## Cloud Storage

In this module, we cover storage and database services in GCP. Every application needs to store data, whether it's business data, media to be streamed, or sensor data from devices.  
From an application-centered perspective, the technology stores and retrieves the data. Whether it's a database or an object store is less important than whether that service supports the application's requirements for efficiently storing and retrieving the data given it its characteristics.  

Google offers several data storage services to choose from. In this module, we will cover **Cloud Storage, Cloud SQL, Cloud Spanner, Cloud Firestore, and Cloud Bigtable**. Let me start by giving you a high level overview of these different services. This table shows the storage and database services and highlights the storage service type, relational, non-relational, object, and data warehouse, what each service is good for and intended use.  

**BigQuery** is also listed on the right. I'm mentioning the service because it sits on the edge between data storage and data processing. You can store data in BigQuery, but the intended use for BigQuery is **big data analysis and interactive querying**.  
For this reason, BigQuery is covered later in the series. If tables aren't your preference, I also added this decision tree to help you identify the solution that best fits your application. Let's walk through this together.  

First ask yourself, **is your data structured**? If it's not, choose Cloud Storage.  
If your data is structured, **does your workload focus on analytics**?  
If it does you will want to choose Cloud Bigtable or BigQuery, depending on your latency and update needs.
Otherwise, check whether your data is relational.  
If it's not relational choose Cloud Firestore.  
If it is relational, you will want to choose Cloud SQL or Cloud Spanner depending on your need for horizontal scalability.  

Depending on your application, you might use one or several of these services to get the job done. For more information on how to choose between these different services see the links section of this video.  
Before we dive into each of the data storage services let's define the scope of this module. The purpose of this module is to explain which services are available and when to consider using them from an infrastructure perspective.  
I want you to be able to set up and connect to a service without detailed knowledge of how to use a database system. If you want a deeper dive into the design, organizations, structures, schemas, and details on how data can be optimized,served, and stored properly within those different services, I recommend Google Cloud's data engineering courses.  
Let's look at the agenda. This module covers all of the services we have mentioned so far. To become more comfortable with these services, you will apply them in two labs. I'll also provide a quick overview of Cloud Memorystore, which is GCP's fully managed Redis service.
Let's get started by diving into Cloud Storage.

**Cloud Storage** is Google Cloud's **object storage service**, and it allows worldwide storage and retrieval of any amount of data at anytime. You can use Cloud Storage for a range of scenarios including serving website content, storing data for archival and disaster recovery, or distributing large data objects to user via direct download.  
Cloud Storage has a couple of key features. It's scalable to exabytes of data. The time to first byte is in milliseconds.  
It has very high availability across all storage classes and it has a single API across those storage classes.   
Some like to think of Cloud Storage as files in a file system, but it's not really a file system.  
Instead, Cloud Storage is a **collection of buckets** that you place objects into. You can create directory, so to speak, but really directory is just another object that points to different objects in the bucket. You're not going to easily be able to index all of these files like you would in a file system. You just have a specific URL to access objects.  

Cloud Storage has four storage classes. **Standard, nearline, coldline, and archive** in each of those storage classes provide three location types. There's the multi region which is a large geographic areas such as the United States that contains two or more geographic places. Dual region is a specific pair of regions such as Finland and the Netherlands.  
A **region** is a specific geographic place such as London. Object stored in a multi region or dual region are geo redundant.  

Now let's go over each of the storage classes.  
**Standard** storage is fest for data that is frequently accessed. Think of hot data. And are stored for only brief periods of time. This is the most expensive storage class, but it has no minimum storage duration and no retrieval cost. When used in a region, standard storage is appropriate for storing data in the same location as Google, Kubernetes engine clusters or compute engine instances that use the data. Co locating your resources maximizes the performance for data intensive computations and can reduce network charges. When used in a dual region, you still get optimized performance when accessing Google Cloud products that are located in one of the associated regions. But you also get improved availability that comes from storing data in geographically separate locations. When used in multi region, standard search is appropriate for storing data that is accessed around the world. Such as serving website content, stream videos, executing interactive workloads, or serving data supporting mobile and gaming applications.  

**Nearline** storage is a low cost-, highly durable storage service for storing infrequently accessed data like data backup, long tailed multimedia content, and data archiving. Nearline storage is a better choice than standard storage in scenarios were slightly lower availability, authority day, minimum storage duration, and costs for data access are acceptable tradeoffs for lowered at less storage costs.  

**Coldline** storage is a very lowcost-, highly durable storage service for storing infrequently accessed data. Coldline storage is a better choice than standard storage or nearline storage. In scenarios where slightly lower availability, a 90 day minimum storage duration and higher costs for data access are acceptable tradeoffs for lowered address storage costs.  

**Archive** storage is the lowest cost, highly durable storage service for data archiving, online backup and disaster recovery. Unlike the source, big coldest storage service offered by other cloud providers, your data is available within milliseconds, not hours or days. Unlike others, cloud storage classes, archive storage has no availability SLA. Though the typical availability is comparible to nearline and coldline storage. Archive storage also has higher costs for data access and operations as well as a 365 day minimum storage duration. Archive storage is the best choice for data that you plan to access less than once a year.  

Let's focus on durability and availability. **All of these storage classes have 11 9s of durability**, but what does that mean? Does that mean that you have access to your files at all times? No, what that means is you won't lose data. You may not be able to access the data, which is like going to your bank and saying, well, my money is in there. It's 11 9s durable. But when the bank is closed, we don't have access to it, which is the availability that differs between storage classes and the location type.  
Cloud storage is broken down into a couple of different items here.  

First of all, there are buckets which are required to have a globally unique name and cannot be nested. The data that you put into those buckets are objects that inherit the storage class of the bucket, and those objects could be text files, Doc files, video files, etc. There's no minimum size to those objects, and you can't scale this as much as you want, as long as your quota allows it. To access the data you can use the gsutil command or either JSON or XML APIs.  
When you upload an object to a bucket, the object is assigned til bucket storage class unless you specify a storage class for the object. You can change the default search plus if a bucket, but you can't change the location type from regional to multi region slash dual region or vice versa.  
You can also change the storage class of an object that already exists in your bucket without moving the object to a different bucket or changing the URL to the object. Setting a pair object storage class is useful. For example, if you have objects in your buckets that you want to keep, but you don't expect to access frequently. In this case, you can minimize costs by changing the storage class of those specific objects to nearline, coldline or archive storage.  

In order to help manage the classes of objects in your bucket cloud storage, offers Object Lifecycle Management, more on that later. Let's look at access control for your objects and buckets that are part of a project. We can use IAM for the project to control which individual users or service account can see the bucket. List the objects in the bucket, view the names of the objects in the bucket or create new buckets.  
**For most purposes Cloud IAM is sufficient and roles are inherited from project to bucket to object**. Access control lists or ACLs offer finer control for even more detailed control.  

**Signed URL** provide a cryptographic key that gives tying limited access to a bucket or object. Finally, a **signed policy document** further refines the control by determining what file can be uploaded by someone with a signed URL.  
Let's take a closer look at ACLs and signed URLs. An ACL is a mechanisms used to define who has access to your buckets and objects, as well as what the level of access to have. The maximum number of ACL entries you can create for a bucket or object is 100.  

Each ACL consists of one or more entries, and these entries consists of two pieces of information.  
**A scope** which defines who can perform the specified actions. For example, a specific user or group of users.  
And **the permission** which defines what actions can be performed. For example, read or write. The allUsers identifier listed under slide represents anyone who is on the Internet, with or without a Google account. The allAuthenticatedUsers identifier, in contrast, represents anyone who is authenticated with a Google account.  

For more information at ACLs, refer to the links of this video. For some applications it is easier and more efficient to grant limited time access tokens that can be used by any user instead of using account based authentication for controlling resource access. For example, when you don't want to require users to have a Google account.  
Signed URLs allow you to do this for cloud storage. You create a URL, the grants read or write access to a specific cloud storage resource, and specifies when this access expires.  
That URL is signed using a **private key associated with their service account**. When the request is received, Cloud Storage can verify that the axis granting URL was issued on behalf of a trusted security principle.  
In this case, the service account and delegates its trust of that account to the holder of the URL. After you give out these signed URL, it is out of your control. So you want these signed URL to expire after some reasonable amount of time.

### Cloud Storage Features

There are also several features that come with Cloud Storage. I will cover these at a high level for now because we will soon dive deeper into some of them. Earlier in the core series, we already talked a little bit about customer-supplied encryption keys when attaching persistent disks to virtual machines.  
This allows you to supply your own encryption keys instead of the Google-managed keys, which is also available for Cloud Storage.  
Cloud Search also provides Object Lifecycle Management, which lets you automatically delete or archive objects.  

Another feature is **Object Versioning**, which allows you to maintain multiple versions of objects in your pocket. You are charged for the versions as if there are multiple files, which is something to keep in mind.  
Cloud Storage also offers **directory synchronization** so that you can sync a VM directory with a bucket.  
e will discuss object change notification, data import, and strong consistency in more detail after going into Object Versioning and Object Lifecycle Management. In Cloud Storage, objects are immutable, which means that an uploaded object cannot change throughout its storage lifetime.  

To support the retrieval of objects that are deleted or are written, Cloud Storage offers the Object Versioning feature. **Object Versioning can be enabled for a bucket**. Once enabled, Cloud Storage creates an archived version of an object each time the live version of the object is overwritten or deleted. The archive version retains the name of the object, but is uniquely identified by a generation number as illustrated on this slide by g1.  
When Object Versioning is enabled, you can list archived versions of an object, restore the live version of an object to an older state, or permanently delete an archived version as needed. You can turn versioning on or off or a bucket at anytime.  

Turning versioning off leaves existing object versions in place and causes the bucket to stop accumulating new archived object versions. For more information on Object Versioning, refer to the link section of this video. To support common use cases like setting a time to live for objects, archiving older versions of objects, or downgrading storage classes of objects to help manage costs, Cloud Storage offers Object Lifecycle Management.  
You can assign a lifecycle management configuration to a bucket. The configuration is a set of rules that apply to all objects in the buckets. When an object meets the criteria of one of the rules, Cloud Storage automatically performs a specified action on the object.  
Here are some example use cases.  
- First, downgrade the storage class of objects older than a year to Cloud line storage.  
- Second, delete objects created before a specific date, for example, January 1st, 2017.  
- Third, keep only the three most recent versions of each object, any bucket with versioning-enabled.

**Object inspection** occurs in asynchronous batches, so rules may not be applied immediately.  
Also, updates to your lifecycle configuration may take up to 24-hours to go into effect. This means that when you change your lifecycle configuration, Object Lifecycle Management may still perform actions based on the old configuration for up to 24-hours, so keep that in mind. For more on Object Lifecycle Management, refer to the link section of this video.  
**Object change notification **can be used to notify an application when an object is updated or added to a bucket through a watch request. Completing a watch request creates a new notification channel. The notification channel is the means by which the notification message is sent to an application watching a bucket. As of this recording, the only type of notification channels supported is a webhook. After a notification channel is initiated, Cloud Search notifies the application at any time an object is added, updated, or removed from the bucket.  

For example, as shown here, when you add a new picture to a bucket, an application can be notified to create a thumbnail. However, pop-ups notifications are really the recommended way to track changes to objects in your Cloud Storage buckets because they're faster, more flexible, easy to set up, and more cost-effective. **Pop-ups is Google's distributed real-time messaging service**, which is covered in the developing applications courses.  
The Cloud Console, allows you to upload individual files to a bucket. But what if you have to upload terabytes, even petabytes of data?  

There are three services that address this: **Transfer Appliance, Storage Transfer Service, and Offline Media Import**.  
**Transfer Appliance** is a hardware appliance you can use to securely migrate large volumes of data from hundreds of terabytes up to one petabyte to Google Cloud without disrupting business operations. The images on the slide are Transfer Appliances.  
**The Storage Transfer Service** enables high-performance imports of online data. The data source can be another Cloud Storage bucket, an Amazon S3 bucket, or an HTTP, HTTPS location.  
Finally, **offline media import** is a third-party service, where physical media such as storage arrays, hard disk drives, tapes, and USB flash drives is sent to a provider who uploads the data.  

For more information on these three services, refer to the links section of this video. When you upload an object to Cloud Storage and you receive a success response, the object is immediately available for download and metadata operations from any location where Google offers service. This is true whether you create a new object or overwrite an existing object.  
Because uploads are strongly consistent, you will never receive a 404 NOT Found response or stale data for a read-after-write or read-after-metadata-update operation. Strong global consistency also extends to the deletion operation of objects. If a deletion request succeeds, an immediate attempt to download the object or its metadata will result in a 404 Not Found status code. You get the 404 error because the object no longer exists after the delete operation succeeds.  
Bucket listing is strongly consistent. For example, if you create a bucket, then immediately perform a list bucket operation, the new bucket appears in the returned list of buckets.  

Finally, object listing is also strongly consistent. For example, if you upload an object to a bucket and then immediately perform a list object operations, the new object appears in the returned list of objects.

### Choosing a storage class

Let's explore the decision tree to help you find the appropriate storage class in Cloud storage.  
If you will read your data less than once a year, you should consider using **Archive Storage**.  
If you will read your data less than once per 90 days, you should consider using **Coldline Storage**.  
If you read your data less than once per 30 days, you should consider using **Nearline Storage**.  
If you'll be doing reads and writes more often that, you should consider using **Standard Storage**.  

You also want to take into account the location type, use a region to help optimize latency and network bandwidth for data consumers, such as analytics pipelines that are grouped in the same region. Use a dual-region when you want similar performance advantages as regions.  
But you also want the high availability that comes with being geo-redundant. Use a multi-region when you want to serve content to data consumers that are outside of the Google network, and distributed across large geographic areas, or when you want the higher data availability that comes with being geo-redundant.

### Lab Review: Cloud Storage

In this lab, you learn to create and work with Buckets and Objects and apply the following Cloud Storage features, Customer Supplied Encryption Keys, Access Control Lists, Life-cycle Management, Object Versioning, Directory Synchronization, and Cross-Project Resource Sharing using IAM. Now that you're familiar with many of the advanced features of Cloud storage, you might consider using them in a variety of applications that you might not have previously considered. A common, quick, and easy way to start using GCP is to use Cloud storage as a backup service. You can stay for a lab walk-through, but remember that GCP's user interface can change. So your environment might look slightly different. Welcome to the walk-through of the Cloud Storage Lab. At this point, I've already started the lab in Qwiklabs and I am logged into the GCP console using the username and password that was provided by Qwiklabs for me to log in to the GCP console. So the first task is preparation. I'm going to create a bucket in here. When I go to create a bucket, it specifically tells me that I should be using a globally unique ID. So I'm going to use my project ID, which is pretty unique. I'm going to call it myproj- and then my project ID, and it's telling us multi-regional. So storage class is multi-regional, and then it's telling me access control is set object-level and bucket-level permissions, and I'm going to hit create. So at this point, you can now go back to the lab page, and you can hit check my progress, and you should get a check mark in five points that you created the Cloud Storage Bucket. Next step is downloading a file. So I'm going to start Cloud Shell so I can do the curl command, and the first thing I'm going to do is I'm going to set an environment variable to the bucket name of the bucket I just created, just for ease of copy paste of commands. Export bucket name one equals and the bucket name. If I want to verify that that worked, I'm going to do an echo dollar sign, and the variable name to make sure that it got set correctly, and there it is. So now I'm going to download a file, which is just a publicly available Hadoop documentation, HTML file, and if I do an ls, I can see there's my setup.html and I am now going to copy it a couple times to make a setup two and a setup three. If I do an ls, I should see three files. There they are. So the second task is ACLs. We're going to copy this file into the bucket and then configure the access control list for it. So the first one is gsutil command, where I am copying setup.html into my bucket.
Once it's copied, I then want to get the default access list that has been assigned to setup.html, which is based on the bucket because that's how we set it. Then right here, I piped it into acl.txt, and now I'm going to cut that, and we can see all of the permissions that had been assigned. So now I want to set the permissions to private. So I'm going to set it to private, and then in order to see it, I'm going to pipe it into acltwo.txt, and then cut that file, and you can see it's now set to private. Update the access list to make the file publicly readable by running the following command, and then I'm going to pipe it into aclthree, so that I can verify what that looks like. You can see it is readable by all users. This is another check point in a lab where you can hit check my progress and in this case is checking if you properly made that file publicly readable. So now I'm going to verify in my bucket using the console that my file is there and that is publicly viewable, and you can tell that based on this little icon and the public link that says that it's accessible to the public.
So now in Cloud Shell, I'm going to remove the setup.html in my local Cloud Shell Instance. There it is. Let me remove it from the search here. If I do an ls, I'll see setup two and setup three, but not setup. You can see it got deleted. Let's say I accidentally deleted that from my Cloud Shell Instance, but now I want the copy that was in the bucket back on my local Cloud Shell. So I could just copy from the bucket to my local Cloud Shell, and if I do an ls again, I'll see all three setup files. There they are. The third task is to generate a customer supplied encryption key. To create the key, I'm going to run this command, and that's going to give me some output, and then I can copy this. But first, I'm going to see if I have a boto file. I'm going to do ls-al, and I do not see a boto file. So what I'm going to do is I'm going to run gsutilconfig-n,
and then I'm going to do ls-al, and I should now see a boto file. There it is. So I'm going to do a nano.boto, and then I'm going to find the encryption key field, which I'm going to exit back out because I didn't not copy the key that I created, which I need. That is right here. Let me copy that, and let me go back to nano, and let me find the line with encryption underscore key.
Could need to expand this because it's very hard to see.
See decryption key here is encryption key. I'm going to uncomment this, and then I'm going to paste in my key here.
I'm going to press control l, write that file, and then control x to exit nano. So now that I've set that up, I am going to upload the remaining setup two and setup three into the bucket. There's one, and there's the other. Now back in the console, let's go down, I'm going to refresh the bucket. I can see both of these files, and it shows that they are encrypted by a customer supply key. So this is another opportunity to check my progress and make sure I got the points for doing that step. Now what I'm going to do is I am deleting my local files by running remove setup star. So it's going to delete setup, setup two, and setup three. Now I am going to copy the files down from the bucket again,
and if I want to cut the encrypted files to see whether I need them back, you can see there they are, and I successfully was able to bring them back even though they're encrypted. So now I'm going to move the current customer supplied encryption key to the decrypt key. So let's go to nano.boto. I'm going to find the comment out the line that I added earlier. I should've noted the line number, so that I wouldn't have to find it again.
Decrypt keys in the GSUtil section. Let's see.
I think I'm close.
I'm looking for that line. So I'm going to comment out encryption key line and uncomment decryption key one right there. Then, I'm going to copy this into decryption key one,
and then we save x. So a best practices you would actually delete the old customer key from the encryption line. But in this case, we just copy pasted it. So it's not a big deal. So I'm going generate a new key and then I'm going back to boto files.
So I am going to add a new encryption key line,
make sure that I copied the new key I made, and then do the same thing again.
Sparsed it so I am adding a new encryption key equals, and I'll paste in the new key. Then control O to save, control X to exit. Now, I'm going to rewrite the key for file 1, and comment out the old decrypt key.
Again, to bottom. Then I am going to comment out the decryption key 1.
Now, while the instructions have you using nano, you definitely could use the Cloud Shell editor as well. That might be a little more pleasant than using this tool, but I'll leave it to you. You would just access that by hitting this little pencil here.
It's fine. Decryption key 1 real quick. So we're commenting that out. Then, we're going to save it, and click. Now, we're going to download setup 2,
and download setup 3.
What happened, no decryption key matches because we commented it out, which makes sense. So the last task in this lab is, we are going to run the following command to view the current life-cycle policy. So we're going to do this.
It says it has no life-cycle configuration. So I'm going to create a JSON lifecycle policy file. I'm going to paste the following rule in here. So it's saying if it's over 31 days I'm going to delete it. Writing exit. Then, to set the policy I'm going to run the command provided in the box,
and to verify that the policy worked, I'm going to press that. This is another opportunity for you to check your progress and get more points in the lab. This point you should have about 20 out of 35 points. The task 6 is enabling versioning and you can do that by using the following command.
Says it suspended, which means it's not enabled. So if we want to enable versioning, we're going to run this command. Then if we were to run the Get-Command again, we would not see that it was suspended we would say that it was enabled. There it is. So check your progress again you'll get more points. The next step, we're going to create several versions of the sample file in the bucket. So I'm going do an ls here. Going to open the setup HTML file. Delete any five lines to change the size. So I'm going to comment out this link and then I'm going to delete all of these links.
Probably a faster way to do this thing just holding down delete. That's what I'm doing here. I'm going to delete it all the way to the banner.
So I have now effectively changed the size of the file. So I'm going to control O, enter, control X. I'm going to copy the file to the bucket.
I'm going to go back to setup.html, delete another five lines.
Let's delete some more links.
I'm just going to delete up to here. I'm going to save it. Then I'm going to copy it again.
So if I wanted to list all versions of the file, which each subsequent one I was deleting different lines and making the size smaller, I was creating a new version. You can see there are three versions: the original one, the one where I deleted the first five lines, and then the one where I deleted the next set of lines. So I am now going to store the version value in the environment variables. So I'm going to say, export version name equals, the oldest version is this one. I'm going to copy that. I'm going to set this variable here make sure it got set correctly, and it is set correctly. Now, I'm going to download the oldest version, call it recovery.text.
Then I'm going to verify recovery with a couple of commands. It is saying, c ls setup.html.
Looks like that piece didn't work. I think what I did was I set the version name to the wrong thing, it should have been here.
So now I can do the Gsutil again,
and it still didn't match.
So you do this, it's because you didn't follow instructions like me. You should have copied the entire URL for that object.
Usually, what happens with the lab is if you have an issue is usually not that the lab is broken. It's usually that you missed a step. So go back three steps and repeat, and that usually works, because you can see here that just worked. Ls.al Setup.html, there's the file. I want to see the recovered text. You can see that the size is different here. So task 7, we're going to synchronize a directory to a bucket and just copy these in. Then I'm going to sync the first-level directory on the VM with my bucket.
I'm going to verify that versioning was enabled.
How I can check in the browser, I'm going to refresh the bucket, and we go back here, first level. You can see there's a second level. We can see the same thing in the console as we do in the command line. So I can exit Cloud Shell. So now we're going to do some cross-project sharing, this is the last little piece of this lab. So I'm going to open another tab. I'm also going to go to console.cloud.google.com,
and I am now signed in, I'm going to select the other project. This one I have 26.
I am going to copy the project from the Qwiklabs site, I'm here at lab guide, and I'm going to select that project. This is my other project.
Then I am going to now create a bucket for this project. There shouldn't be one in here because it's a new project. I'm going to also call it myproj in project ID, and Create. This will now be bucket name two. So I'm going to upload a file, any file.
I've uploaded a screenshot, and this will be my file name. So I'm actually going to rename it. I can't.
Now I'm going to go to IAM, service accounts, and then create a service account.
I'm going to call it cross-project-storage, and click create. Then I'm going to give it the Storage Object Viewer. Click continue. I'm going to create a key. I'm going to select JSON, create, and then it's going to download that file for me. It's there. Hit close, and I can hit done. So I am now going to rename this credentials.json.
Here it is. I'm going to switch back to the other project, check my progress, and I should get five more points. So now we are just five points away from finishing the lab. Now we are in project ID one, and we're going to create a VM, create. Calling it crossproject. I'm going to make it in Europe in D, and I'm making it a micro, and create.
VM is ready, I'm going to SSH into it.
There it is, click SSH, then move my window back here
to get the bucket name of the project I created here.
I'm going to verify that it worked,
and then I'm going to export the file name of the file that I uploaded.
Grab that, put quotes around there because that space is in it.
Verify that worked, and there it is. LS what's in that bucket. Form a VM on this side, it tells me that I don't have access to do that, so now I'm going to verify that. I am going to upload here, upload file. I'm going to select the credentials.json that I downloaded. Close, then I'm going to authorize that file to verify access. I'm going to do this again, and now I can see my file in there.
I can do it with the file as well. Let me try to copy these credentials so they don't have access to that project. So if I wanted to do that, I would go back to this project, and modify the role in IAM. It Should be my last step here, going back to IAM, cross-project-storage, pencil. I'm also going to give Storage Object Admin, save. Once I hit save, I can check my progress,
and then you will have all of the points in the lab. The last step is optional, you're just going to return to your SSH terminal, and verify that everything is good to go but that is the entire walkthrough for this lab. I hope you enjoyed it.

## Cloud SQL

Let's dive into the structured or relational database services. First up is Cloud SQL. Why would you use a GCP service for SQL, when you can install SQL server application image on a VM using Compute Engine?  
The question really is, should you build your own database solution or use a managed service? **There are benefits to using a managed service**. So, let's learn about why you'd use Cloud SQL as a managed service inside of GCP.  

Cloud SQL is a fully managed service of either **MySQL or PostgreSQL** databases. This means that patches and updates are automatically applied. But you still have to administer MySQL users with the native authentication tools that come with these databases.  
Cloud SQL supports many clients, such as Cloud Shell, App Engine and G Suite scripts. It also supports other applications and tools that you might be used to using like SQL Workbench, Toad and other external applications using standard MySQL drivers.  
Cloud SQL delivers high performance and scalability eith up to 30 terabytes of storage capacity, 40,000 IOPS, and 415 GB or RAM per instance. You can easily scale up to 64 processor core s and scale out with read replicas.  
You can use Cloud SQL with either MySQL 5.6 or 5.7, or PostgreSQL 6.9 or 11.1 as of this recording. Let's focus on some other services provided by Cloud SQL. There is a replica service that can replicate data between multiple zones as shown on the right. This is useful for automatic failover if an outage occurs. Cloud SQL also provides automated and on demand backups with point in time recovery.  

You can import and export databases using MySQL dump or import and export CSV files. Cloud SQL can also scale up, which does require a machine restart or scale out using read replicas. That being said, if you're concerned about horizontal scalability, you'll want to consider cloud spatter, which we'll cover later in this module. Choosing a connection type to your Cloud SQL instance will influence how secure, performant, and automated it will be.
If you are connecting an application that is hosted within the same GCP project as your Cloud SQL instance, and it is co located in the same region. Choosing the private IP connection will provide you with the most performant and secure connection using private connectivity. In other words, traffic is never exposed to the public Internet.  

If the application is hosted in another region or project, or if you are trying to connect to your Cloud SQL instance from outside of GCP, you have three options.  
In this case, I recommend using Cloud Proxy, which handles authentication, encryption and key rotation for you. If you need manual control over the SSL connection, you can generate and periodically rotate the certificates yourself.  
Otherwise, you can use an unencrypted connection by authorizing a specific IP address to connect to your SQL Server over its external IP address. You will explore these options in the upcoming lab. But if you want to learn more about private IP, see the link section of this video.  

To summarize, let's explore this decision tree to help you find the right data storage service with full relational capability.  
If you need more than 30 terabytes of storage space or over 4,000 concurrent connections to your database. Or if you want your application design to be responsible for scaling, availability and location management when scaling up globally. Then consider using **Cloud Spanner**, which we will cover later in this module.
If you have no concerns with these constraints, ask yourself whether you have specific OS requirements, custom database configuration requirements or special backup requirements.  
If you do - consider **hosting your own database** on a VM using Compute Engine.  

Otherwise, I strongly recommend using Cloud SQL as a fully managed service for your relational databases. If you're now convinced that using Cloud SQL as a managed service is better than using or re implementing your existing MySQL solution. See the link section for a solution on how to migrate from MySQL to Cloud SQL?

### Lab Review: Cloud SQL

In this lab you created a Cloud SQL database and configured it to use both an external connection over a secure proxy and a private IP address, which is more secure and performant. Remember, you can only connect via private IP if the application and the Cloud SQL Server are co-located in the same region and are part of the same VPC network. If your application is hosted in another Region, VPC, or even project, use a proxy to secure its connection over the external connection. You can stay for a lab walkthrough, but remember that GCP's user interface can change, so your environment might look slightly different.
Welcome to the lab walkthrough for implementing Cloud SQL. At this point in the lab, I have logged in with the username and password that QwikLabs has provided me. My first task is to create a Cloud SQL database. And go here to SQL, And hit Create Instance. I'm going to choose my SQL,
And I'm going to call this wordpress-db. For password I'm just going to use the word password, so that I make sure I'm not forgetting it. I recommend you use something very simple. I'm going to use US Central 1. I am going to expand the configuration options, and then in Connectivity I'm going to select the private IP. Hit Enable API, and once that's been enabled, which could take a couple of seconds, I'm going to hit Allocate and Connect. And just so you know, this could take three to five minutes, so just be patient. Once that's done, it is going to make this Create button enabled. Feel free to look through some of the other things that it's calling out in the lab, like configuring the machine type and changing the storage capacity. If you add a couple of zeros, you can see the throughput increases. Set it back to 10, hit Close here. Again, this could take three to five minutes, so be patient. Once it's done, it'll say Create here.
All right, so my IP has been allocated, and now I can hit Create.
Here we go. And now, that took a while, but creating your Cloud SQL instance or Cloud SQL instance, might take even longer. So be patient, but while this is creating I can do other steps in the lab.
The verification in step 15 is going to require that your Cloud SQL instance is running, and that there is a green check here. So you won't be able to get that step and those five points until this is done, but you can do some of the other steps while we wait. So while this is going, I'm going to open another tab so that this is still running and I can check on it. And I am now going to go to Compute Engine, which I could either go here, or this tile right here has my Compute Engine instances. As you can see, two have been created for me.
wordpress-europe-proxy, which is the proxy for my Cloud SQL instance and for the private IP instance. So for this one, I'm going to click SSH.
And when that is ready, to SSH into I am going to download the Cloud SQL proxy and then I'm going to make it executable.
So I'm going to do a wget, Enter. It's copied it and made it executable. So in order to start the proxy, you need the connection name of the Cloud SQL instance, which requires that it's actually running. So we'll go ahead, and go back here, and see what the status is. I'm going to refresh and see if anything is there.
Now it's not there, but it does let me click on it. And let's see if the connection name is there, and it is here. Instance connection name, so I'm going to copy that, and I'm going to go back to my SSH window.
And I am going to create an environment variable for that connection instance. So I'm going to do an export SQL_CONNECTION =, paste that in there. And if I want to verify that that environment variable is set, I'm going to do an echo SQL, oops, spell it right.
And it should output that, and there it is. So I'm not going to get too far ahead, because the rest of the steps do require that my Cloud SQL instance is running. You can see it's still creating, so it is not going to let me create a database yet. So I'm just going to leave it on this screen, so that when it is completely done it'll let me create a database, which I need for the next step.
All right, at this point my instance is running. So if I go to Cloud SQL, it'll have a green checkmark next to wordpress-db. Right here, it says it's runnable. So the step I was missing is I need to create a database, and I am going to create a database called wordpress, because that's what the application expects, and I'm going to hit Create.
And now I am going to return to, My SSH window, and I'm just going to make sure that it still has my environment variable. Then I am going to activate the proxy connection to my SQL database. By running this, I'll run it in the background. And then I'm going to expect that it says Ready for new connections, which it has output. I'm going to press Enter. And this is a point in the lab where you can also hit Check my Progress. And at this point you should have all ten points in the lab, but we're still going to do one more step, which is task 3. Actually, task 3 and task 4, where we're going to connect the application to the Cloud SQL instance. So I'm going to configure the wordpress application. So I'm going to copy the curl command. Go ahead and run that, and it is going to output the external IP address for my virtual machine. I'm going to copy that, and open it here, And then I'm going to hit Let's Go. I am going to leave everything with default, except I'm going to change the username to root and I'm going to put in the password that I defined, which is password. And then for database host I am going to use a localhost IP, which is 127.0.0.1, and then I'm going to hit Submit.
Now, when the connection has been made, it is going to let me install WordPress, and I'm going to click Run the Installation. This could take a few moments to complete. Once this is done I should get a success window, and this can take up to three minutes depending on where you are running your lab from. So here it goes, apparently it's very fast for me. And so once the connection has been made, I am going to go here, and I am going to remove everything passed to the external IP. Delete, hit Enter.
And when this loads, I should be able to see my blog.
So it looks like it's still installing, so I will go ahead and be patient. Add some Information that I don't need to remember. So this is My Fake Site title. Any username, leave that password it gave me, m@b.com, and Install. And this was actually a step in the lab that I ignored, which was step 7, so don't do what I did and skip a step.
So I had a successful installation. So what I'm going to do here is I'm going to remove all of the information that's after the external IP. When I hit Enter, it should take me to my blog. And Hello world! This is my blog, success. So the last task is to connect to Cloud SQL via the internal IP. So I'm going to go back here, and go to SQL and gcp. I'm going to click on wordpress-db, and then I am going to note the private IP address here. And I'm actually going to note it on a note, because pretty sure that it's going to have me copy something else. So make sure you copy it down somewhere in a clipboard, and then I'm going to go to Compute Engine. And it is going to want me to copy the external IP address for WordPress private IP. I'm going to copy that, and paste it in a new tab, press Enter. I'm going to hit Let's Go, and then the database name I'm going to leave alone, and I'm going to change this to root, and leave password because that's what I put before. Then I'm going to put the SQL private IP that I copied earlier to my Notepad, just need to find it. It's here, I'm going to copy that in here and hit Submit. And then I'm going to hit Run the Installation, and I should get Already Installed. So I created a direct connection to a private IP instead of configuring a proxy, and that connection is private. If I remove here, the same private IP should get my blog, and there it is. So in review, we created a Cloud SQL database and we configured it to use an external connection over a secure proxy as well as a private IP address. Hope you enjoyed the lab, thanks for watching.

## Other Database Services

### Cloud Spanner

If Cloud SQL does not fit your requirements because you need horizontal scalability, consider using **Cloud Spanner**.  
Cloud Spanner is a service built for the Cloud, specifically to **combine the benefits of relational database structure with non-relational horizontal scale**. This service can provide petabytes of capacity and offers transactional consistency at global scale, schemas, SQL, and automatic synchronous replication for high availability. Use cases include financial applications and inventory applications, traditionally served by relational database technology. Depending on whether you create a multi-regional or regional instance, you'll have different monthly up-time SLAs as shown on this slide.  

However, for up-to-date numbers, you should always refer to the documentation which you'll find in the link section of this video.  
Let's compare Cloud Spanner with both relational and non-relational databases.  
Like a relational database, Cloud Spanner has **schema, SQL, and strong consistency**.  
Also like a non-relational database, Cloud Spanner offers **high availability, horizontal scalability, and configurable replication**.  

As mentioned, Cloud Spanner offers the best of the relational and non-relational worlds. These features allow for mission-critical use cases such as building consistent systems for transactions and inventory management in the financial services in retail industries.  
To better understand how all of it works, let's look at the architecture of Cloud Spanner.  

Cloud Spanner instance replicates data in n Cloud zones, which can be within one region or across several regions. The database placement is configurable. Meaning, you can choose which region to put your database in.  
This architecture allows for high availability and global placement. The replication of data, will be synchronized across zones using Google's global fiber network. Using atomic clocks, ensures atomicity whenever you are updating your data. That's as far as we're going to go with Cloud Spanner. Because the focus of this module, is to understand the circumstances when you would use Cloud Spanner.  

Let's look at a decision tree.  
If you have **outgrown any relational database** or sharding your databases for **throughput** high-performance, need **transactional consistency**, global data, and strong consistency, or just want to **consolidate** your database. Consider using Cloud Spanner.  
If you don't need any of these nor full relational capabilities, consider a **NoSQL** service such as **Cloud Firestore** which we will cover next. If you're now convinced that using Cloud Spanner as a managed service is better than using or re-implementing your existing MySQL solution, see the link section for a solution on how to migrate from MySQL to Cloud Spanner.

### Cloud Firestore

If you are looking for a highly scalable NoSQL database for your applications, consider using Cloud Firestore.  
Cloud Firestore is a fast, fully managed, serverless, cloud native, NoSQL document database that simplifies storing, syncing, and querying data for your mobile, web, and IoT apps at global scale. Its client libraries provide live synchronization and offline support and its security features and integrations with Firebase and GCP accelerate building truly serverless apps.  

Cloud Firestore also supports **ACID** transactions, so if any of the operations in the transaction fail and cannot be retried, the whole transaction will fail. Also with automatic multi-region replication and strong consistency your data is safe and available even when disasters strike.  
Cloud Firestore even allows you to run sophisticated queries against your NoSQL data without any degradation in performance. This gives you more flexibility in the way you structure your data.  

Cloud Firestore is actually the next generation of Cloud Datastore. Cloud Firestore can operate in Datastore mode, making it backwards compatible with Cloud Datastore. By creating a Cloud Firestore database in Datastore mode, you can access Cloud Firestore's improved storage layer while keeping Cloud Datastore system behavior. This removes the following Cloud Datastore limitations. Queries are no longer eventually consistent. Instead they are all strongly consistent. Transactions are no longer limited to 25 entity groups. Rights to an entity group are no longer limited to one per second.  

Cloud Firestore in native mode introduces new features such as a new strongly consistent storage layer. A collection and document data model. Real-time updates. Mobile and web client libraries. Cloud Firestore is backward compatible with Cloud Datastore, but the new data model real-time updates in mobile and web client library features are not. To access all of the new Cloud Firestore features, you must use **Cloud Firestore in native mode**.  

A general guideline is to use Cloud Firestore in data store mode for new server projects and native mode for new mobile and web apps. As the next generation of Cloud Datastore, Cloud Firestore is compatible with all cloud Datastore APIs and client libraries. Existing Cloud Datastore users will be live upgraded to Cloud Firestore automatically at a future date. For more information, see the link section of this video.  
To summarize, let's explore this decision tree to help you determine whether Cloud Firestore is the right storage service for your data. If your **schema might change** and you need an adaptable database, you need to scale to 0. Or you want low maintenance overhead scaling up to terabytes, consider using Cloud Firestore.  
Also, if you don't require transactional consistency, you might want to consider Cloud Bigtable, depending on the cost or size. I will cover cloud BigTable next.

### Cloud Bigtable

If you **don't require transactional consistency**, you might want to consider Cloud Bigtable.  
Cloud Bigtable is a fully managed NoSQL database with petabyte-scale and very low latency. It seamlessly scales for throughput, and it learns to adjust to specific access patterns.  
Cloud Bigtable is actually the same database that powers many of Google's core services including Search, Analytics, Maps, and Gmail.  

Cloud Bigtable is a great choice for both **operational and analytical applications including IoT, user analytics, and financial data analysis** because it supports high read and write throughput at low latency. It's also a great storage engine for **machine learning** applications.  
Cloud Bigtable integrates easily with popular big data tools like Hadoop, Cloud Dataflow, and Cloud Dataproc, plus Cloud Bigtable supports the open-source industry standard HBase API, which makes it easy for your development teams to get started.  

Cloud Dataflow and Cloud Dataproc are covered later in the course series. For more information on the HBase API, see the link section of this video. Cloud Bigtable stores data in massively scalable tables, each of which is a sorted key value map.  
The table is composed of rows, each of which typically describes a single entity, and columns which contain individual values for each row. Each row is indexed by a single row key. Columns that are related to one another are typically grouped together into a column family. Each column is identified by a combination of the column family and a column qualifier which is a unique name within the column family. Each row column intersection can contain multiple cells or versions at different timestamps, providing a record of how the stored data has been altered over time.  

Cloud Bigtable tables are **sparse**. If a cell does not contain any data, it does not take up any space. The example shown here is for a hypothetical social network for United States presidents, where each president can follow posts from other presidents. Let me highlight some things.  
The table contains one column family, the follows family. This family contains multiple column qualifiers. Column qualifiers are used as data.  
This design choice takes advantage of the sparseness of Cloud Bigtable tables and the fact that new column qualifiers can be added as your data changes. The username is used as the row key. Assuming usernames are evenly spread across the alphabet, data access will be reasonably uniform across the entire table. This diagram shows a simplified version of Cloud Bigtables overall architecture. It illustrates that processing which is done through a front end server pool and nodes is handled separately from the storage.  
A Cloud Bigtable table is sharded into blocks of contiguous rows called tablets, to help balance the workload of queries. Tablets are similar to HBase regions, for those of you who might have used the HBase API. Tablets are stored on Colossus which is Google's file system in SSTable format. An SSTable provides a persistent, ordered, immutable map from keys to values where both keys and values are arbitrary byte strings.  

As I mentioned earlier, Cloud Bigtable learns to adjust to specific access patterns. If a certain Bigtable node is frequently accessing a certain subset of data, Cloud Bigtable will update the indexes so that other nodes can distribute that workload evenly as shown here.  
That throughput scales linearly, so for every single node that you do add, you're going to see a linear scale of throughput performance up to hundreds of nodes.  

In summary, if you need to store more than one terabyte of structured data, have very high volumes of writes. Need read write latency of less than 10 milliseconds along with strong consistency or need a storage service that is compatible with the HBase API. Consider using Cloud Bigtable.  
If you don't need any of these and are looking for a storage service that scales down well, consider using Cloud Firestore. Speaking of Scaling, the smallest Cloud Bigtable cluster you can create has three nodes and can handle 30,000 operations per second. Remember that you pay for those nodes while they are operational, whether your application is using them or not.

### Cloud Memorystore

Let me give you a quick overview of Cloud Memorystore.  
**Cloud Memorystore for Redis** provides a fully managed in-memory data store service built on scalable, secure, and highly available infrastructure managed by Google. Applications running on GCP can achieve extreme performance by leveraging the highly scalable available secure Redis service without the burden of managing complex reddest deployments. This allows you to spend more time writing code, so that you can focus on building great apps. Cloud Memorystore also automates complex tasks like enabling high availability, failover, patching, and monitoring.  

High availability instances are **replicated across two zones** and provide a 99.9 percent availability SLA. You can easily achieve this sub-millisecond latency and throughput your applications need. Start with the lowest tier and smallest size and then grow your instance effortlessly with minimal impact to application availability.  
Cloud Memorystore can support instances of up to 300 gigabytes and network throughput of 12 gigabytes per second.  
Because Cloud Memorystore for Redis is fully compatible with the **Redis** protocol, you can lift-and-shift your applications from open-source Redis to Cloud Memorystore without any code changes by using the import export feature. There is no need to learn new tools because all existing tools in Client Libraries just work.

## Resource Management

In this module, we will cover resource management. Resources and GCP are billable, so managing them means controlling cost. There are several methods in place for controlling access to the resources and there are quotas that limit consumption. In most cases, the default quotas can be raised on request. But having them in place provides a checkpoint or a chance to make sure that this really is a resource you intend to consume in greater quantity.  

In this module, we will build on what we learned in the Cloud IAM module. First, I will provide an overview of the resource manager. Then, we will go into quotas, labels and names. Next, we will cover billing to help you set budgets and alerts. To complete your learning experience, you will get to examine billing data with BigQuery in a lab.
Let's get started with an overview of resource manager.

### Cloud Resource Manager

The resource manager lets you hierarchically manage resources by project, folder, and organization. This should sound familiar because we covered it in the Cloud IAM module. Let me refresh your memory. Policies contain a set of roles and members, and policies are set on resources. These resources inherit policies from their parent as we can see on the left. Therefore, resource policies are a union of parent and resource.  
Also, keep in mind that if a parent policy is less restrictive, it overrides the more restrictive resource policy.  
Although IAM policies are inherited top to bottom, billing is accumulated from the bottom up, as we can see on the right.  
Resource consumption is measured in quantities like rate of use or time, number of items, or feature use. Because a resource belongs to only one project, a project accumulates the consumption of all its resources.  

Each project is associated with **one billing account**, which means that an organization contains all billing accounts.  
Let's explore organizations, projects, and resources more.  
Just to reiterate, an **organization node** is the root node for all Google Cloud Platform resources. This diagram shows an example where we have an individual Bob, who has control of the organizational domain through the organization admin role. Bob has delegated privileges and access to the individual projects to Alice by making her a project creator.  
Because a **project accumulates the consumption of all its resources**, it can be used to track resources and quota usage. Specifically, projects that you enable billing, manage permissions and credentials, and enabled service and APIs.  

To interact with Cloud Platform resources, you must provide the identifying project information for every request.  
A project can be identified by the project name, which is a human-readable way to identify your projects, but it isn't used by any Google APIs.  
There's also the project number, which is automatically generated by the server and assigned to your project, and there is the Project ID, which is a unique ID that is generated from your project name.  
You can find these three identifying attributes on the dashboard of your GCP console, or by querying the Resource Manager API.  

Finally, let's talk about the **resource hierarchy**.  
From a physical organization standpoint, resources are categorized as **global, regional, or zonal**. Let's look at some examples.  
**Images, snapshots, and networks, are global resources**.  
**External IP addresses are regional resources**, and **instances and disks are zonal resources**. However, regardless of the type, each resource is organized into a project. This enables each project to have its own billing and reporting.

### Quotas

Now that we know that a project accumulates the consumption of all its resources, let's talk about quotas.  
All resources and GCP are subject to **project quotas or limits**. These typically fall into one of the three categories shown here. How many resources you can create per project?  

For example, you can only have five VPC networks for project. How quickly you can make API requests in a project or rate limits. For example, by default, you can only make five administrative actions per second per project when using the Cloud Spanner API.  
There are also **regional quotas**. For example, by default, you can only have 24 CPUs per region. Given these quotas, you may be wondering, how do I spin up one of those 96 core VMs? As your use of GCP expands over time, your quotas may increase accordingly. If you expect a notable upcoming increasing usage, you can proactively request quota adjustments from the quotas page in the GCP Console. This page will also display your current quotas.  

If quotas can be changed, why do they exist? Project quotas prevent runaway consumption in case of an error or malicious attack.  
For example, imagine you accidentally create a 100 instances instead of 10 Compute Engine instances using the gcloud command-line. Quotas also prevent billing spikes or surprises. Quotas are related to billing, but we will go through how to set up budgets and alerts later, which will really help you manage billing.  

Finally, quotas forces consideration and periodic review.  
For example, do you really need a 96 Core instance? Or can you go with a smaller and cheaper alternative? It is also important to mention the quotas are the maximum amount of resources you can create for that resource type as long as those resources are available. Quotas do not guarantee that resources will be available at all times. For example, if a region is out of local SSDs, you cannot create local SSDs in that region even if you still had quota for local SSDs.

### Labels and names

Projects and folders provide levels of segregation for resources.  
But what if you want more granularity? That's where **labels and names** come in.  

Labels are a utility for organizing GCP resources.  
Labels are key value pairs that you can attach to your resources like VMs, disks, snapshots, and images. You can create and manage labels using the GCP Console, gcloud or the Resource Manager API, and each resource can have up to 64 labels. For example, you could create a label to define the environment of your virtual machines. 

Then you define the label for each of your instances as either production or test. Using this label, you could search and list all of your production resources for inventory purposes. **Labels can also be used in scripts** to help analyze costs or to run bulk operations on multiple resources. The screenshot on the right shows an example of four labels that are created on an instance. 

Let's go over some examples of what to use labels for. I recommend **adding labels based on team or cost center** to distinguish instances owned by different teams. You can use this type of label for cost accounting or budgeting. For example, team colon marketing and team colon research. 

You can also **use labels to distinguish components**. 
For example, component colon redis, component colon front end. Again, you can **label based on environment or stage**. You should also consider using labels to define an owner or a primary contact for a resource. For example, owner colon gaurav, contact colon OPM or add labels to your resources to define their state. For example, state colon in-use, state colon ready for deletion. 

Now it's important to **not confuse labels with tags**. 

**Labels**, we just learned, are **user-defined strings in key-value format** that are used to organize resources and they can propagate through billing. 

**Tags**, on the other hand, are user-defined strings that are **applied to instances only** and are mainly used for networking such as applying firewall rules. For more information about using labels see the link section of this video.

### Billing

Because the consumption of all resources under a project accumulates into one billing account, let's talk billing. To help with project planning and controlling costs. You can set a budget.

 Setting a budget lets you track how your spend is growing towards that amount. This screenshot shows the budget creation interface. First, you set a budget name and specify which project this budget applies to. Then you can set the budget at a specific amount or match it to the previous month spend. 

After you determine your budget amount, you can **set the budget alerts**. These alerts send emails to billing admins after spend exceeds a percentage of the budget or a specified amount. In our case, it would send an email when spending reaches 50 percent, 90 percent and a 100 percent of the budget amount. 

You can even choose to send an alert when the spend is forecasted to exceed the percent of the budget amount by the end of the budget period. In addition to receiving an email, you can use **Cloud Pub Sub notifications** to programmatically receive spend updates about this budget. 
You could even create a **Cloud Function** that listens to the Pub Sub topic to automate cost management. For an example of programmatic budget notifications, see the link section of this video. Here's an example of an email notification. The email contains the project name, the percent of the budget that was exceeded, and the budget amount. 

Another way to help optimize your GCP spend is to **use labels**. For example, you could label VM instances that are spread **across different regions**. Maybe these instances are sending most of their traffic to a different continent which could incur higher cost. In that case, you might consider relocating some of those instances or using a caching service like Cloud CDN to cach content closer to your users, which reduces your networking spend. 

I recommend labeling all your resources and **exporting your billing data to BigQuery** to analyze your spend. BigQuery is Google's scalable, bully managed enterprise data warehouse with SQL and fast response times. Creating a query is as simple as shown in this screenshot, which you will explore in the upcoming lab. 
You can even **visualize spend over time with Data studio**. Data Studio turns your data into an formative dashboards and reports that are easy to read, easy to share, and fully customizable. For example, you can slice and dice your billing reports using your labels.

### Lab Review: Examining Billing Data with BigQuery

In this lab, you imported billing data into BigQuery that had been exported as a CSV file. You first ran a simple query on that data, next you accessed a shared dataset containing more than 22,000 records of billing information, you then ran a variety of queries on that data to explore how you can use BigQuery to gain insight into your resources billing consumption. If you use BigQuery on a regular basis, you'll start to develop your own queries for searching out where resources are being consumed in your application. You can also monitor changes in resource consumption over time. This kind of analysis is an input to capacity planning and can help you determine how to scale up your application to meet growth or scale down your application for efficiency. Welcome to the walk-through of the lab examining billing data with BigQuery. At this point in the lab, I have logged in with the username and password that Qwiklabs has provided me from the lab. So the first task is to use BigQuery to import data. So what I did is as the Billing Administrator, I exported my billing data and put it in a bucket. So I am going to go into BigQuery and I'm going to import some stuff. So BigQuery?

Yes. The Cloud Council. Thank you.
Make sure that you are logged in to BigQuery and have the correct Qwiklabs project ID selected at the top. So I'm going to go here and I'm going to click Create dataset and I am going to call it imported billing data. My data location is in the US and I want it to expire one day afterwards. I'm going to hit Create dataset.
You can see my dataset is created and I should see it right here. There it is. So now I'm going to create a table in that dataset. 

Table. For the source, I'm going to use Cloud Storage, I'm going to copy and paste the bucket location from the lab and it is in CSV format. For destination, I'm using that and native table and I am going to call it sampleinfotable. Under schema, I I'm going to hit auto detect so it detects a schema and input parameters from the dataset. I'm going to open up the advance and I am going to specify that I want to skip one row because that's the headers, and then I'm going to hit Create table. So this is a point where you can check the progress in your lab. If you click check my progress, it's going to check that you have dataset at a table and that you have imported that data into that table, and you should get five points for that. 

So Task 2, you are going to examine the data that you've just input. So I'm going to click on my table and it's going to, by default, show the schema. I can click Details and it's going to tell me a little bit more information about number of rows. You can see it has 44 rows, it's a pretty small table. I can hit Preview and it's going to show me a couple of the first rows of the table. So it now wants me in the lab. There are some formative questions that are going to ask you just to make sure that you're understanding the learning. So I'm not going to go over that in the walk-through because that is more and to make sure that you're understanding what you're doing. 

So I'm going to go to Task 3 where we're composing a simple query. A couple of cool things about BigQuery, if you are in a table by default, if you click Query table, it's going to auto-populate the query editor with the project dataset table for you and then you just specify what you want to look at. So I do select star and I'm doing this. Oops. Where cost is greater than zero. So I just want to see in this table how much of it. I only want to see the rows where the cost is more than zero. You can see right here it's validating that my SQL or my SQL is right. I'm going to hit Run. Here are my query results and you can see out of a table that has 44, there are 20 rows in this table that actually have costs that are more than zero. So again, there are a couple more questions that you can answer and you can also check your progress that you've run this query. 

So if you run the query, then it's going to give you another five points. At this point, you're actually done with the points that are awarded in the lab, but you still have another task to go into a little more complex query. So I'm going to go ahead and copy the query from Task 4, and I'm going to erase this and paste it in. It's a valid query here. I'm going to hit Run, then I'm going to verify that the result has what my lab is telling me is, supposed to return 22,537 lines of billing data. I can see right here, that is correct. Let's say I wanted to find the latest 100 records where the charges were greater than zero. So I'm going to copy paste the query that's provided to me, and make sure it's valid. Always good to check that your SQL is valid. Hit Run, and it is going to show me the last 100 records where charges were greater than zero. Let's say I wanted to find all of the charges that were more than `$3`, the next query shows you that. You can feel free to click through each one of these more complex queries and feel free to try out some queries of your own. If you wanted to just peruse the data and figure out maybe the last two days of billing anything that was over `$10`. Any kind of question that you might need to provide data for to your senior leadership about your billing of resource usage in GCP.

After all of these complex queries, in review, you imported billing data that was technically exported for you from a billing admin into BigQuery, and then you run a simple query and then you ran some more complex queries. I hope you enjoyed the lab. Thank you.

### Google Cloud's Operations Suite

Stackdriver dynamically discovers cloud resources and application services, based on deep integration with Google Cloud Platform and Amazon Web Services. Because of its smart defaults, you can have core visibility into your cloud platform in minutes. This provides you with access to powerful data and analytics tools. Plus collaboration with many different third-party software providers. 

As I mentioned earlier, Stackdriver has services for **monitoring, logging, error reporting, fault tracing and debugging**. You only pay for what you use. And there are free usage allotments, so that you can get started with no upfront fees or commitments. For more information about Stackdriver pricing, see the link section of this video. Now, in most other environments these services are handled by completely different packages, or by a loosely integrated collection of software. When you see these functions working together in a single, comprehensive, and integrated service, you'll realize how important that is. To creating reliable, stable, and maintainable applications. Stackdriver also supports a rich and growing ecosystem of technology partners, as shown on this slide. This helps expand the ITOps, security, and compliance capabilities available to GCP customers. For more information about Stackdriver integrations, see the link section of this video.

## Monitoring

Now that you understand Stackdriver from a high-level perspective, let's look at Stackdriver monitoring. 

Monitoring is important to Google because it is **at the base of site reliability engineering**, or SRE. 
SRE is a discipline that applies aspects of software engineering to operations whose goals are to create ultra scalable and highly reliable software systems. This discipline has enabled Google to build, deploy, monitor, and maintain some of the largest software systems in the world. If you want to learn more about SRE, I recommend exploring the free book written by members of Google's SRE team. It is in the link section of this video. 

Stackdriver **dynamically configures monitoring after resources are deployed** and has intelligent defaults that allow you to easily create charts for basic monitoring activities. 
This allows you to monitor your platform, system and application metrics by ingesting data, such as **metrics, events, and metadata**. You can then generate insights from this data through dashboards, charts, and alerts. For example, you can configure and measure uptime and health checks that send alerts via e-mail. 

A **workspace** is the root entity that holds monitoring and configuration information in Stackdriver monitoring. Each workspace can have between one and 100 monitored projects, including one or more GCP projects, and any number of AWS accounts. You can have as many workspaces as you want, but GCP projects and AWS accounts can be monitored by more than one workspace. 
A workspace contains the **custom dashboards, alerting policies, uptime checks, notification channels, and group definitions** that you use with your monitored projects. A workspace can access metric data from its monitored projects, but the **measured data in log entries remain in the individual projects**. The first monitor GCP project in a workspace is called the hosting project and it must be specified when you create the workspace. The name of that project becomes the name of your workspace. To access an AWS account, you must configure a project in GCP to hold the **AWS connector**. Because workspaces can monitor all of your GCP projects in a single place, a workspace is a single pane of glass through which you can view resources from multiple GCP projects and AWS accounts. 

All Stackdriver users who have access to that workspace **have access to all data by default**. This means that a Stackdriver role assigned to one person on one project applies equally to all projects monitored by that workspace. 
In order to give people different roles per project and to control visibility to data, consider **placing the monitoring of those projects in separate workspaces**. Stackdriver monitoring allows you to create custom dashboards that contain charts of the metrics that you want to monitor. For example, you can create charts that display your instances CPU utilization. The packets are bytes sent and received by those instances, and the packets are bytes dropped by the firewall of those instances. 
In other words, charts provide visibility into the utilization and network traffic of your VM instances, as shown on this slide. These charts can be customized with **filters** to remove noise, groups to reduce the number of time series and aggregates to group multiple time series together, for a full list of supported metrics, see documentation linked for this video. 
Now, although charts are extremely useful, they can only provide insight when someone is looking at them. But what if your server goes down in the middle of the night or over the weekend. Do you expect someone to always look at dashboards to determine whether your servers are available or have enough capacity or bandwidth? 

If not, you want to create **alerting policies** than notify you when specific conditions are met. For example, as shown on this slide, you can create an alerting policy when the network egress of your VM instance goes above a certain threshold for a specific time-frame. When this condition is met, you or someone else can be automatically notified through e-mail, SMS, or other channels in order to troubleshoot this issue. 
You can also create an alerting policy that monitors your Stackdriver usage and alerts you when you approach the threshold for billing. For more information about this, see the link section of this video. Here's an example of what creating an alerting policy looks like. On the left, you can see an HTTP check condition on the summer01 instance. This will send an e-mail that is customized with the content of the documentation section on the right. 

Let's discuss some best practices when creating alerts. I recommend **alerting on symptoms** and not necessarily causes. For example, you want to monitor failing queries of a database and then identify whether the database is down. Next, make sure that you're using multiple notification channels, like e-mail and SMS. This helps avoid a single point of failure in your alerting strategy. 
I also recommend **customizing your alerts** to the audiences need by describing **what actions need** to be taken or what resources need to be examined. Finally, avoid noise because this will cause alerts to be dismissed overtime. Specifically adjust monitoring alerts so that they are actionable and don't just set up alerts on everything possible. 

**Uptime checks** can be configured to test the availability of your public services from locations around the world. As you can see on this slide, the type of uptime check can be set to HTTP, HTTPS, or TCP. The resource to be checked can be an App Engine application, a Compute Engine instance, a URL of a host or an AWS instance, or load balancer. For each uptime check, you can create an alerting policy and view the latency of each global location. Here is an example of an HTTP uptime check. The resources checked every minute with a 10 second timeout. Uptime checks that do not get a response within this time out period are considered failures. So far there is 100 percent uptime with no outages. 

Stackdriver monitoring can access some metrics without the monitoring agent, including CPU utilization, some disk traffic metrics, network traffic and uptime information. However, to access additional system resources and application services, you should **install the monitoring agent**. The monitoring agent is supported for Compute Engine and EC2 instances. The monitoring agent can be installed with these two simple commands which you could include in your startup script. 

This assumes that you have a VM instance running Linux that is being monitored by a workspace and that your instance has the proper credentials for the agent. If the standard metrics provided by Stackdriver monitoring do not fit your needs, you can create custom metrics. For example, imagine a game server that has a capacity of 50 users. What metric indicator might you use to trigger Scaling events? From an infrastructure perspective, you might consider using CPU load or perhaps network traffic load as values that are somewhat correlated with the number of users. But **with a custom metric**, you could actually pass the current number of users directly from your application into Stackdriver. To get started with creating custom metrics, see the link section of this video.

### Lab Review: Resource Monitoring


In this lab you got an overview of stackdriver monitoring. You learned how to monitor your project, create alerts with multiple conditions, add charts to dashboards, create resource groups, and create uptime checks for your services. Monitoring is critical to your applications health. And Stackdriver provides a rich set of features for monitoring your infrastructure, visualizing the monitoring data and triggering alerts and events for you. You can stay for a lab walkthrough, but remember that GCP is user interface can change, so your environment might look slightly different. Welcome to the lab walkthrough for resource monitoring with stackdriver. At this point I have logged into the GCP console with the credentials that the Quick Labs lab has provided me. And in the first task I am going to verify that the proper Vms had been set up for me using deployment manager in this lab. And as you can see, there are three Vms. 3 engine X stacks right here. So now that I verified that my instances are running and were created for me, I am going to go to stackdriver monitoring which will open in a new tab. And then it is going to set up the workspace for my project. And this can take a few minutes. So just be patient or go get a cup of coffee and come back. Once your workspace has been set up for you, you are going to be redirected to the monitoring overview page. There are some questions in the labs that are going to ask you some questions and those are just making sure that your understanding and actually reading. But you don't have to fill those out in order to get the full score for the labs. So in task two, we're going to create a dashboard, so I'm going to go here. To monitoring overview when I click create dashboard. I am going to name it my dashboard instead of untitled hit enter. And then I'm going to add a chart. For the title, I am going to say this is my chart. And I am going to find GCE. VM instance. For metrics, I am going to select CPU utilization. CPU utilization and for filter. Where is filter here? I'm going to add a filter. There are various options you can filter by resource label by metadata label. I'm not going to add any filters, I want to see everything. And then we click here on view options. There's a couple of chart modes. There's color mode X Ray mode. You can preview it on the right. Stats mode and like that. So I actually like the X Ray mode, so I'm going to go ahead and click that. And then you're going to hit save to add the chart to your dashboard. There it is, looking nice. So then we also have a metrics explorer which allows you to examine resources and metrics without having to create a chart on the dashboard. So if I go to resources metrics explorer. Find resource type in metric, I can type any metric or resource name. So let's say I do CPU utilization. And as you can see I didn't have to add this, but I could still explore it. Again, you'll have another question in the lab, but that is to prompt your understanding. So now, I'm going to create an alert and add the first condition. So I'm going to go here and create a policy. And I'm going to click add condition. Here I'm going to do GCE VM instance. And for metrics, I'm going to use CPU usage. For condition, I'm going to say is above threshold. Of one minute, the threshold is 20. And then I'm going to hit save. And I'm going to add another condition. And then what said do I do it for another VM? So if I do this one. Maybe I do it for another metric going to, do you? Stu reserved course. And then above 15. I'm going to head, save. So now in policy triggers I'm going to trigger when all conditions are met. Then I'm going to configure the notifications so that I can be actually told that this has triggered. And I'm going to click here. Email, and I'm going to add some email going to do fake email. I want you guys spamming me. And then add. That has been added. And then I'm going to stick, skip the documentation step, but in reality this is a pretty important part of your notification. You want to say. What happened, why whoever it is that's getting notified is being alerted and a best practices to actually tell them how to maybe fix it. because otherwise that's not a very useful notification if they don't know how they can fix it. And then you're going to name it, and this is my. I first alerting policy. And then I'm going to hit save. This is a checkpoint in the lab where you can check your progress that you have created an alerting policy. The next task you are going to create some groups here. Create group, I'm going to give it a name. VM instances. Name, I am going to select. Contains and I'm going to type engine X. And I'm going to save group. And you can see it's showing me instances. All three of my instances because the name matches Anjanette stack. And again, you're going to have another. Question to make sure that your understanding and reading all of the extra tidbits that are available in the lab. So now, we're going to go back to the Dashboard.
You' re going to go to uptime checks. Overview and then we're going to add an uptight and check. So in here we're going to add a title, so my first uptime check were using HTTP. It is to check an instance in, applies to a group and I'm going to select group I created which is VM instances and I'm going to check every one minute. I'm going to hit save. You could also hit test for it and make sure that it works. I'm going to say no thanks. I don't want to create the alert policy right now. So this is the last piece where you can hit check my progress. It's going to make sure that you created that uptime check, and if so, you're going to get the full points for the lab. So in this lab, we got to walk through monitoring your projects, creating a stackdriver workspace which gets created for you. Creating some alerts with multiple condition, adding some charts to a dashboard, and creating resource groups, and finally we created an uptime check for your services. Hope you enjoyed it.

## Logging, Error Reporting, Tracing and Debugging

### Logging

Monitoring is the basis of Stackdriver, but the service also provides logging error reporting, tracing and debugging. 
Let's learn about logging, Stackdriver Logging allows you to **store search, analyze, monitor, and alert on log data and events from GCP and AWS**. It is a fully managed service that performs at scale and can ingest application and system log data from thousands of EMS. 

Logging include **storage** for logs, a **user interface** called the log viewer, and an **API** to manage logs programmatically. The service let's you read and write log entries, search and filter your logs and create log based metrics. Logs are only r**etained for 30 days**, but you can **export your logs to cloud storage buckets, BigQuery datasets and cloud Pub/Sub topics**. 

Exporting logs to cloud storage makes sense for storing logs for more than 30 days, but why would you export to BigQuery or cloud Pub/Sub? Exporting logs to BigQuery allows you to analyze logs and even visualize them in data studio. BigQuery runs **extremely fast SQL queries** on gigabytes to petabytes of data. This allows you to **analyze logs such as your network traffic** so that you can better understand traffic growth to forecast capacity. Network usage to optimize network traffic expenses, or network forensics to analyze incidents. For example, in this screenshot I queried my logs to identify the top IP addresses that have exchange traffic with my web server. Depending on where these IP addresses are and who they belong to, I could relocate part of my infrastructure to save on networking costs. Or deny some of these IP addresses if I don't want them to access my web server. 

If you want to visualize your logs, I recommend **connecting your BigQuery tables to data studio**. Data Studio transforms your raw data into the metrics and dimensions that you can use to create easy to understand reports and dashboards. I mentioned that you can also **export logs to Cloud Pub/Sub**, this enables you to **stream logs to applications or end points**. Similar to Stackdriver monitoring agent, it's a best practice to install the **logging agent** on all of your VM instances. The logging agent can be installed with these two simple commands which you could include in your startup script. This agent is supported for compute engine and EC2 instances.

## Error Reporting


Let's learn about another feature of error reporting. Stackdriver Error Reporting counts, analyses, and aggregates the errors in your running Cloud services. A centralized error management interface displays the results with sorting and filtering capabilities. You can even set up real-time notifications when new errors are detected. In terms of programming languages, the exception stack trace parser is able to process Go, Java,.NET, Node.js, PHP, Python, and Ruby. By the way, I'm mentioning App Engine because you will explore error reporting in an app deployed to App Engine in the upcoming lab.

### Tracing

**Tracing** is another Stackdriver feature integrated into GCP. Stackdriver Trace is a distributed tracing system that collects latency data from your applications and displays it in the GCP console. You can track how request propagate through your application and receive detailed near real time performance insights. Stackdriver Trace automatically analyzes all of your applications traces to generate in depth latency reports that surface performance degradations, and can capture traces from App Engine, HTTPS load balancers, and applications instrumented with the Stackdriver Trace API. Managing the amount of time it takes for your application to handle incoming requests and perform operations is an important part of managing overall application performance. Stackdriver Trace is actually based on the tools used at Google to keep our services running at extreme scale.

### Debugging

Finally, let's cover the last Stackdriver feature of this module, which is the **debugger**. Stackdriver debugger is a feature of GCP, that lets you inspect the state of a running application in real time without stopping or slowing it. Specifically, the debugger adds **less than 10 milliseconds to the request latency** when the application state is captured. In most cases, this is not noticeable by users. These features allow you to understand the behavior of your code in production and analyze its state to locate those hard to find bugs. With just a few mouse clicks, you can take a snapshot of your running application state or inject a new logging statement. Stackdriver debugger supports multiple languages, including **Java, Python, Go, Node.js, and Ruby**.

### Lab Review: Error Reporting and Debugging

In this lab you diploid an application to App Engine. Then you introduced a bug in the code which broke the application, used stackdriver error reporting to identify and analyze the issue, and found the root cause using stackdriver debugger. Finally, you modified the code to fix the problem and saw the results in Stackdriver. Having all of these tools integrated into GCP allows you to focus on your code and any troubleshooting that goes with it. You can stay for a lab walkthrough, but remember that GCP is user interface can change. So your environment might look slightly different. Welcome to the error reporting an debugging with Stackdriver lab walkthrough. I have logged into the GCP console with the username and password Quick Labs has given me and I am going to start Cloud Shell. You can always hide or show the navigation menu by hitting this icon up here. So the first thing I'm going to do is I'm just going to create a local folder and get the Hello World Application so that I can deploy it to app engine. So to run the application using the local development server in Cloud Shell, I'm going to run the following command.
And then once it is running.
I am going to, it'll tell me that is ready for me to preview it.
All right, I'm going to hit Web Preview, preview on Port 88.
And in theory I should get a Hello, World and there it is. In order to stop this, I'm going to hit control C. And we're back at the command prompt. So now that I know that my app works, I am going to deploy the application to App Engine by running gcloud app deploy and then the YAML file for my app.
Add to do I'm going to enter my choice for a region and I am going to select 15 which is uswest2. And then this is going to take a minute or two to run.
Once the process is done, which shouldn't take very long.
I'm going to browse to my app.
Do I want to continue? Yes. And it's deploying, and see it uploaded for files to Google Cloud Storage as part of deploying to App Engine.
Just doing some traffic splitting for the service and you can see it's giving me a bunch of details about how it's deploying my app. And it will tell you right here the same command that we're going to run, which is gcloud app browse. And then it is going to give me a URL here which I should be able to click, and there we go. Hello, World, same as the one that I ran locally on my Cloud Shell. This is a point in the lab where you can check your progress and make sure you get 5 out of 10 points in a green check mark that you properly deploy your app to App Engine. So I'm going to go back to Cloud Shell and I'm going to hit control C just to make sure, I'm going to cat the main PY file. Make sure that I see what's in there. And, what you want to notice is that it's importing web app too. So that is what it's requiring. So what I'm going to do is I'm going to use the said stream editor so that I'm going to embed a bug, so I'm going to replace webapp2 with webapp22. Cause I know that doesn't exist and that is going to cause an error in my app. You can see here it's replacing webapp2 with webapp22. Enter, and then I can verify that that happened by cutting the file again and there it is. So now I'm going to redeploy the app to App Engine because I changed it. I'm going to use the quiet flag, which is going to disable all those interactive prompts of like, do you want to continue, and just say yes to everything.
Now, once that's done again, it's going to output the command I have to run, which is gcloud app browse. And when I click here, I'm assuming it's not going to work. There's an error because there is no webapp22.
So if needed you'll press control C, but we don't need to. You will have some questions that are prompting your understanding of the lab. I'm not going to fill those out, I'll let you do that to make sure your understanding what you're learning. And you can hit check my progress, which should give you another five points and you should at this point at the end of task one, get a full 100 points for the lab, test two, and three are more exploration. So now let's explore Stackdriver Error Reporting. So we introduced a bug and we should be able to see that bug. So I'm going to go to Error Reporting, We are down here.
So we should see an error which is the same error that we saw here, no module named webapp22. There we see it, we hit auto reload, and then in Cloud Shell, we're going to do gcloud app browse. And go back to it.
So if we click here on the URL.
We're going to create another error.
And then let's just click that link a couple times to see if we can generate the air a few more times and make sure it's auto reloading. And here you can see now it has two errors, no it has five. We can see, we've looked at 1, 2, 3, 4, 5 internal server errors and it's showing 5 here in Stackdriver Error Reporting. So let's say I click the error. It's going to show me a graph of how many times this error happened, what time, date, lots of details about it. So Stack Trace Sample, so we're going to find that. And in Parsed, we click Parsed, it's going to show me the exact line where there's a mistake, which is really nice. If I click here, it's going to take me to Stackdriver debug. So that I could technically view the logs and fix it.
So I could, see the logs, it's going to show me the actual logs.
I can introduce more errors you can see here at each time that it didn't work.
I am going to go back here and I'm going to fix my code. So I am going to go back and replace webapp22 with webapp2. I am going to redeploy my app. It's the quiet flag, so it doesn't brought me again.
And then once that has been redeployed, as you can see, the first time you deploy tap, it takes a little bit longer and then the subsequent ones are a little bit faster. When I go back to browse, go back to the page and we should see Hello, World cause no one has the right one.
And now I could go back into debugger, I shouldn't see anymore errors, so let's go back to.
We were to go back to Error Reporting, we saw five and then they stopped because we fix the code. So as you can see with Stackdriver, we were able to notice exactly what was happening with our app, when we got an error, we click on it and live debug it while the app is still running. Hope you enjoyed this lab walkthrough.

