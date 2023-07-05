---
layout: post
title: My Journey to CKA Certification
categories:
- k8s
tags:
- tips
image:
  path: https://www.cncf.io/wp-content/uploads/2022/05/cka-share.png
---
# Objectives

Throughout this post, I aim to provide you with a comprehensive account of my CKA certification experience. 

From the initial preparations to the final examination, I'll share the tips, resources, and strategies that proved invaluable in navigating the intricate Kubernetes ecosystem. 

Whether you're a seasoned developer seeking to enhance your skill set or a budding professional aspiring to make your mark in the industry, this series aims to equip you with the knowledge and confidence to embark on your own CKA journey.


# Preparation

## Essential Tools to Have in Your Backpack

When preparing for the exam, having the right tools at your disposal is crucial. While knowledge is paramount, being equipped with useful tools can greatly enhance your ability to tackle challenges efficiently. Here are some tools that you should have in your backpack:

1. ViM: Familiarize yourself with ViM and learn useful YAML settings. It's a powerful text editor that can streamline your workflow.

2. Bash: Being comfortable with the command line is essential. Brush up on your Bash skills to navigate and execute commands effortlessly.

3. TMUX: While not mandatory for the exam, TMUX is a handy tool for managing terminal sessions. It can help improve your productivity during day-to-day work.

4. Kubectl: Considered your go-to tool, Kubectl is essential for managing Kubernetes clusters. Practice using it extensively from day one to become proficient.

5. Containers: Docker knowledge is indispensable. Make sure you understand how containers work, their lifecycle, and how to interact with them effectively.

6. CRICTL: If you're familiar with Docker commands, you'll find CRICTL to be very useful. Its commands are similar and can provide valuable insights into container runtimes.

7. JSON: Become proficient in using tools like jq or mastering jsonpath. These will enable you to manipulate and extract information from JSON data efficiently.

8. Openssl: Understanding how to check the validity of certificates using Openssl is important. It will help you troubleshoot and secure communication in a Kubernetes environment.

9. SystemD: Gain familiarity with systemctl, daemon units, and journalctl. SystemD plays a vital role in managing services, so having a good grasp of its commands and logs is beneficial.

By ensuring you have these essential tools in your arsenal, you'll be well-prepared to handle the challenges that come your way during the exam. Remember to practice using them regularly to build confidence and proficiency. Good luck with your exam preparation!

# Embarking on the Journey: A Strategic Approach

## Navigating the Route

To make the most of your journey, I highly recommend starting from day zero by actively engaging with a Kubernetes cluster. There are several options available to you:

* minikube (highly recommended): This local Kubernetes cluster is an excellent choice for experimentation and learning.

* Vagrant: Explore the possibility of setting up a cluster using Vagrant, which provides a convenient way to create and manage virtual machines.

* GCP Free-Tier: Utilize the free-tier offering from Google Cloud Platform (GCP) to deploy and interact with a Kubernetes cluster in the cloud.

* VMs with proxmox: For a more advanced setup, consider deploying virtual machines using proxmox with Terraform scripts and configuring them via Ansible.

Remember, the method of cluster deployment is not as crucial as the hands-on experience you gain. It's essential to dive into the journey and immerse yourself in various commands and operations.

## Exploring the Route

As you embark on this journey, be prepared to interact extensively with the official Kubernetes documentation available at <https://kubernetes.io/docs/home/>. However, it's important to adopt a focused approach rather than attempting to read every single post. Instead, concentrate on the key and vital concepts.

By honing in on the crucial information, you can efficiently grasp the fundamental concepts and techniques required for success in your Kubernetes endeavors. Prioritize understanding the core concepts, such as Pods, Deployments, Services, and Ingress, while also exploring topics like networking, storage, and security.

Remember, practical experience and hands-on exercises will be your greatest assets as you navigate this journey. Combine your theoretical knowledge with practical implementation to reinforce your understanding and gain confidence in working with Kubernetes.

Wishing you an enlightening and rewarding journey ahead!

**REMEMBER:** You will be allowed to use that documentation on the exam, so get ready to deep dive in that page.

I have prepared the exam with **Mumshad Mannambeth** awesome course, I know others providers but I can tell you, this is pretty clear and concise.

#### Try to go deeper

I recommend you to try to go deep on each topic if needed, try to understand the reasons behind each part. I've used this material:

https://www.youtube.com/@PeladoNerd/videos

https://www.youtube.com/@justmeandopensource

https://www.youtube.com/@cncf

[Securing Cluster Networking with Network Policies - Ahmet Balkan, Google - YouTube](https://www.youtube.com/watch?v=3gGpMmYeEO8&t=7s)

[Kubernetes Best Practices with Sandeep Dinesh (Google) - YouTube](https://www.youtube.com/watch?v=BznjDNxp4Hs)

## Key Knowledge to Acquire

To ensure a comprehensive understanding of Kubernetes, it is important to focus on the following key areas:

* Role-Based Access Control (RBAC): Differentiate between user and service account roles. Understand how RBAC governs access and permissions within a Kubernetes cluster.

* Volumes: Familiarize yourself with the distinction between emptyDir and hostPath volume types. Understand their use cases and implications for data storage within containers.

* Network Policies: Gain a solid understanding of how network policies function and the various approaches to implementing them. Refer to the [GitHub repository by ahmetb](https://github.com/ahmetb/kubernetes-network-policy-recipes) for practical examples and recipes for Kubernetes network policies.

* Taints & Tolerations: Experiment with taints and tolerations to grasp their functionality and comprehend the different types of taints available. Explore how taints and tolerations contribute to node selection and workload scheduling.

* Container Networking Interface (CNI): Develop an understanding of how CNI works within a Kubernetes cluster. Explore how CNI plugins facilitate network communication between containers and nodes.

* ETCD: Familiarize yourself with the backup and restore processes for ETCD, the distributed key-value store that stores Kubernetes cluster state. Understand how to perform backup and restoration both from within a pod and externally.

* Services: Gain in-depth knowledge of the various types of Kubernetes services. Understand how services enable communication between different components within a cluster and the specific use cases they serve.

By mastering these key concepts and techniques, you will establish a strong foundation in Kubernetes and be better equipped to navigate and troubleshoot various scenarios within your cluster.

Remember, practical hands-on experience and continuous learning will further solidify your understanding of these topics. Explore real-world use cases, experiment with different configurations, and seek out additional resources to deepen your knowledge.

Happy learning and may your Kubernetes journey be fruitful!

# The exam

## Preparation

Make sure you have practiced a lot your skills with **kubectl**, try to do everything with that command. 

When you feel secure, take a mock exam from <https://killer.sh>

Take a week to review what it has gone bad for you in that exam and try to grasp the missing points, after that review take another mock exam. You will be a lot more confident.

## The final day

In my case, my mind was a chaos of knowledge but when the proctor assigned my exam and I started answering the questions it all started to become easy. 
If you followed the steps in this post you cannot fail on the first attempt.

Although, I will give you these advices:

* Do not spend too much on each question

* Before resolving anything scan all the questions and select to resolve first the ones with higher score
  
  * For this I used a vim text file to order the questions

# Useful tips

###### Get the JSONPath easily

```bash
# Need the jq utility
k get nodes -o json | jq -c 'paths'
```

###### Display the data in columns

```bash
k get nodes -o=custom-columns=<COL_NAME>:<JSONPATH> --sort-by=<SORT_KEY>
```

###### How to know if user can do certain action

```bash
k auth can-i list persistentvolumes --as system:serviceaccount:default:pvviewer
```
