# COMP3010 - BOTSv3 Incident Analysis & Presentation

## Introduction
In this report I will be covering an investigation I took on with the BOTSv3 dataset using Splunk Enterprise. 

My goal for this investigation was to use Splunk to identify and gather evidence regarding suspicious or misconfigured activity across AWS and such endpoints. 

Following this, I will be documenting my findings in a threat intelligence style report suitable for a SOC and Security Management audience.

My objectives were as such:

•	Set up Splunk Enterprise to digest and confirm the datasets coverage.

•	Identify AWS identity activity and relevant authentication context regarding MFA.

•	Investigate an S3 public access misconfiguration actions associated with it.

•	Identify endpoint anomalies in host OS information.

For my SOC interpretation, and for the basis of this report. I will be answering one set of the level 200 questions provided by the BOTSv3 team, alongside documenting my findings and showing clear and relevant evidence for each question.
## SOC Roles & Incident Handling Reflection
This investigation followed how a typical SOC Team would handle an incident such as the one we were given with the level 200 questions. The investigation made up the three different tiers found in a SOC Team:

•	Tier 1 (Triage Specialist)

•	Tier 2 (Incident Responder)

•	Tier 3 (Threat Hunter)

Tier 1 Triage Specialists are typically involved in collecting raw data and reviewing alerts and telemetry to validate whether there is suspicious activity ongoing, they try to confirm whether they are dealing with a false positive or an actual alert that needed looking into [1]. In the context of my investigation, that looked like spotting the AWS API activity within S3 where BSTOLL accidentally made the bucket publicly available and checking whether it was a real threat or just a false positive.

Tier 2 Incident Responders are more involved with acting and performing further analysis on the alerts to determine its impact and start planning on what they need to contain and begin recovery actions. This was seen with my investigation in Q5-Q7 where I confirmed who it was that changed the S3 bucket permissions, what was exposed, and what happened next which was the uploads to the bucket warning that it was open. Using CloudTrail and S3 Access logs I was able to visualise what was happening and when it was occurring.

Tier 3 Threat Hunters focus on the most critical alerts (such as S3 misconfigurations) where it’s clear that there could be an actual threat if action isn’t taken and they perform deep analysis to improve detection. This stage is where I would follow up to see if any other ACL changes occurred or if there was repeated public access patterns indicating that there were more threats occurring.

By working through the guided questions, I was able to mirror a real SOC workflow (identify, triage, scope, contain, learn) which provided valuable lessons showing that  the investigation wasn’t necessarily about just finding the answers, rather it was learning how to work through the dataset, finding credible evidence to back my findings and understanding the impact and implications misconfigurations or even small accidents may cause within a network if not handled correctly.

## Installation & Data Preparation
To begin the investigation, I had to get a Virtual Machine set up and run the Ubuntu 24.04.3 ISO to run Linux which gave me a consistent and isolated environment in which I could run Splunk locally without affecting my host system while being able to mirror a typical SOC analysis environment.
Following the Ubuntu setup, I then installed and configured Splunk and used Splunk Web to install all the necessary add-ons listed in the BOTSv3 GitHub repo, then validate dataset ingestion properly by running a few SPL queries to make sure everything was properly set up.
By installing the addons to my Splunk, it ensured that the dataset was parsed correctly so that all my searches provided me with reliable results making my evidence and write up stronger.

Add-ons Installed
![image alt]([Github-Addons.png](https://github.com/Chris-Bratosin/COMP3010/blob/97ee4f6da877c6466cfb5a69699c8256b2e9a5eb/Evidence/SPL_INSTALLATION/Github%20Addons.png)
Figure 1 - BOTSv3 GitHub Repo Add-ons
Above is the ‘Required Software’ listed in the BOTSv3 repo which stated all the add-ons I had to install to get all the questions answered.
Splunk Installation 
 
Figure 2 - Splunk.deb file + Accepting Licenses
 
Figure 3 - Creating Splunk Account
 
Figure 4 - Uploading Addon onto Splunk Web
 
Figure 5 - Splunk Web Homepage w/ Addons
 
Figure 6 - Locating dataset in downloads to ingest into Splunk
 
Figure 7 - BOTSv3 Dataset Ingestion
BOTSv3 data was then ingested into the botsv3 index and then ingestion was validated by running the following queries:
 
Figure 8 - Validating BOTSv3 Exists
 
Figure 9 – Sorting Source Types
 
Figure 10 – Dataset Time Range
Running validation checks ensured that the dataset was properly ingested Splunk Web. Figure 8 shows the total amount of Events from the earliest count to the latest and proves that the dataset exists. Figure 9 shows the breakdown of each source type and the count of events they each have. Figure 10 shows the time coverage of the 2,842,010 events spanning from 08/20/2018 04:00:03.00 – 09/19/2019 18:10:50.00


## Conclusion + Recommendations
During this investigation I was able to find multiple indicators of security risks within the BOTSv3 AWS Frothly environment. Using CloudTrail I found that the user bstoll was involved in the modification of an S3 bucket where he made is publicly accessible for READ and WRITE. When looking into the S3 access logs, we saw that there was confirmed uploads using HTTP_PUT to the affected bucket notifying that the bucket was open. Additionally, I was also able to find the FQDN endpoint showing that there was an anomaly host making use of a different Windows Editions within the froth.ly domain which highlighted the misconfigurations further that if this was within a live environment, would cause detection/response issues.

I would recommend that S3 Block Public Access is enabled at the account level alongside enabling restrictions on ACL usage where possible alongside enabling alerts for CloudTrail events that indicate bucket permission being modifying. Finally, I’d recommend implementing stricter MFA, ensuring that it is implemented for all IAM users and alert when mfaAuthenticated returns false. 

## References 
[1] “Security Operations Center (SOC) Roles and Responsibilities,” Palo Alto Networks. https://www.paloaltonetworks.co.uk/cyberpedia/soc-roles-and-responsibilities

[2] AWS, “What Is IAM? - AWS Identity and Access Management,” Amazon.com, 2025. https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html

[3] “CloudTrail log file examples - AWS CloudTrail,” docs.aws.amazon.com. https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-examples.html

[4] “Who turned on public access to S3 bucket | AWS re:Post,” Amazon Web Services, Inc. https://repost.aws/knowledge-center/s3-bucket-public-access

[5] AWS, “What Is AWS CloudTrail? - AWS CloudTrail,” docs.aws.amazon.com, 2024. https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html

[6] “PutBucketAcl - Amazon Simple Storage Service,” docs.aws.amazon.com. https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketAcl.html

[7] “PutObject - Amazon Simple Storage Service,” Amazon.com, 2016. https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutObject.html

## Appendix
