# COMP3010 - BOTSv3 Incident Analysis & Presentation
- Audience: SOC Team Lead/Secuity Management
  
- Evironemnt: Splunk Enterprise 10.0.2 on Ubuntu 24.04.3 (Oracle VirtualBox VM)
  
- Dataset: Splunk BOTSv3 (index=botsv3)
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

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/97ee4f6da877c6466cfb5a69699c8256b2e9a5eb/Evidence/SPL_INSTALLATION/Github%20Addons.png)

Figure 1 - BOTSv3 GitHub Repo Add-ons

Above is the ‘Required Software’ listed in the BOTSv3 repo which stated all the add-ons I had to install to get all the questions answered.
Splunk Installation 

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/ac3a2a9b8c99168cf184fc71da077e10461a1d09/Evidence/SPL_INSTALLATION/Install%20%2B%20Accept%20License.png)
Figure 2 - Splunk.deb file + Accepting Licenses

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/ac3a2a9b8c99168cf184fc71da077e10461a1d09/Evidence/SPL_INSTALLATION/Settings%20Account%20Details.png) 
Figure 3 - Creating Splunk Account

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/ac3a2a9b8c99168cf184fc71da077e10461a1d09/Evidence/SPL_INSTALLATION/Uploading%20Add-on%20to%20SPL.png)
Figure 4 - Uploading Addon onto Splunk Web

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/ac3a2a9b8c99168cf184fc71da077e10461a1d09/Evidence/SPL_INSTALLATION/SPL%20Add-Ons%20Installed.png)
Figure 5 - Splunk Web Homepage w/ Addons

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/ac3a2a9b8c99168cf184fc71da077e10461a1d09/Evidence/SPL_INSTALLATION/Finding%20Data%20Set%20in%20Downloads.png)
Figure 6 - Locating dataset in downloads to ingest into Splunk

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/ac3a2a9b8c99168cf184fc71da077e10461a1d09/Evidence/SPL_INSTALLATION/Ingesting%20the%20Data%20Set.png)
Figure 7 - BOTSv3 Dataset Ingestion
BOTSv3 data was then ingested into the botsv3 index and then ingestion was validated by running the following queries:

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/ac3a2a9b8c99168cf184fc71da077e10461a1d09/Evidence/BOTSv3_INGESTED_VALIDATION/Validating%20Botsv3%20Exists.png)
Figure 8 - Validating BOTSv3 Exists

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/ac3a2a9b8c99168cf184fc71da077e10461a1d09/Evidence/BOTSv3_INGESTED_VALIDATION/Sorting%20Sourcetypes.png)
Figure 9 – Sorting Source Types

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/ac3a2a9b8c99168cf184fc71da077e10461a1d09/Evidence/BOTSv3_INGESTED_VALIDATION/Showing%20Data%20Set%20Time%20Range.png)
Figure 10 – Dataset Time Range

Running validation checks ensured that the dataset was properly ingested Splunk Web. Figure 8 shows the total amount of Events from the earliest count to the latest and proves that the dataset exists. Figure 9 shows the breakdown of each source type and the count of events they each have. Figure 10 shows the time coverage of the 2,842,010 events spanning from 08/20/2018 04:00:03.00 – 09/19/2019 18:10:50.00
## Guided Questions
In this section, I will document how I went about answering the level 200 questions and how I treated each question like a small, evidence-based investigation. My methodologies and SPL queries I used should help to understand my reasoning and answers for each question.
Some questions may have multiple pieces of evidence, as they contain answers to other questions, allowing me to clearly explain my thought process behind what I did.

### Q1 – List out the IAM users that accessed and AWS service (successfully or unsuccessfully) in Frothly’s AWS environment.
Using the aws:cloudtrail source type I was able to filter it to userIdentity.type=IAMUser which helped to identify any of the users that were using IAM to perform any AWS API activities within the Frothly environment.

![alt textt](https://github.com/Chris-Bratosin/COMP3010/blob/fd277268123fd2746f1224ffb89d22dea957680c/Evidence/SPL_Q1_EVIDENCE/Table%20of%20Users.png)

Whilst the first SPL query provided me a table of users making use of IAM, it only returned one username.

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/164642f40fbf5753e4baf0116fb204bbf3a3e90f/Evidence/SPL_Q1_EVIDENCE/Q1_IAM_USERS_ANSWER.png)

To find the full list of users I filtered it to userIdentity.type=”IAMUser” to avoid it outputting the activities by the ‘assumedroles’ ensuring it returned all the users, this was summarised by using stats values(userIdentity.userName) which outputted the list of usernames involved in using IAM within the Frothly environment.

### Q2 – What field would you use to alert that AWS API activity has occurred without MFA?

For Q2, I am asked to figure out what JSON field can be used to indicate the API activity without MFA that is also excluding ConsoleLogin.

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/9a71d2b2e8baccec086f5b7263e691dd91360e8e/Evidence/SPL_Q2_EVIDENCE/Checking%20for%20MFA%20Usage.png)

I started by searching the dataset using aws:cloudtrail with MFA being the search criteria allowing me to search for any events containing the MFA in raw JSON. By using spath I am able to parse the CloudTrail JSON so that fields I am looking for can actually be searched for and selected.
What I found was that I got an output of 2,351 events which in the first 20, the MFA field was blank which meant that majority of them were probably going to be blank. 

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/9a71d2b2e8baccec086f5b7263e691dd91360e8e/Evidence/SPL_Q2_EVIDENCE/Results%20indicating%20MFA%20is%20stored%20differently.png)

I added the | where isnotnull(additionalEventData.MFAUsed) so that it would instead output the events where the field I needed was populated rather than empty. However, there were no fields found with this new filter being added. 
This indicated that the MFA was potentially being stored differently from what I was searching for.

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/9a71d2b2e8baccec086f5b7263e691dd91360e8e/Evidence/SPL_Q2_EVIDENCE/Q2_fieldname_answer.png)

I then used fieldsummary and search field=”*MFA*” to find fields where the MFA was present within the dataset. By doing this, I got two outputs where I found userIdentity.sessionContext.attributes.mfaAuthenticated to be the consistent MFA Indicator with a total of 2155 counts of being used.

### Q3 – What is the processer number used on the web servers?
![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/57acc58c04328f5945e64894f798abf0aaf4d828/Evidence/SPL_Q3_EVIDENCE/Checking%20Hardware%20Souretype%20exists.png)

For Q3, I began by running a sanity check with sourcetype=hardware to confirm that the dataset has any relevant hardware events. This resulted in getting 3 outputs with the hardware source type.

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/57acc58c04328f5945e64894f798abf0aaf4d828/Evidence/SPL_Q3_EVIDENCE/Possible%20Hosts%20.png)

I then needed to figure out what hosts this hardware events belonged to so that I can figure out what CPU types they were using on the systems. 

By running the | stats count by host | sort – count SPL query, I was able to return the list of hosts that the hardware events belonged to. All I had to do now to figure out the CPU type was click on any one of the events and extract the CPU model.

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/57acc58c04328f5945e64894f798abf0aaf4d828/Evidence/SPL_Q3_EVIDENCE/Q3__CPU_TYPE_EVIDENCE.png)

By inspecting the event, I was able to find the CPU model (Intel(R) Xeon(R) CPU E5-2676 v3 @ 2.40GHz) used across the web servers (all 3 hosts were identical).

### Q4 – Bud accidentally made an S3 bucket publicly accessible. What is the event ID of the API call that enabled public access to the S3 bucket?
![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/a73dc95c6e2269e79170a59fbbefbb5759c53063/Evidence/SPL_Q4_EVIDENCE/Finding%20PutBucketAcl%20events.png)

My goal was to determine whether an S3 bucket was made public, but importantly, which S3 bucket was made public and what its eventID was. 

To do this, I used sourcetype=aws:cloudtrail and then filtered it to eventName=PutBucketAcl. Reason being that typically ACL is what appears as what grants full public access to groups like AllUsers or AuthenticatedUsers.

However, this result gave me two events: one at 13:01:46 and one at 13:57:54. To figure out which was the public one, I ran the following:

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/a73dc95c6e2269e79170a59fbbefbb5759c53063/Evidence/SPL_Q4_EVIDENCE/Sorted%20for%20Public%20Event.png)

I filtered it to buckets that had any of the criteria; public, AllUsers, or AuthenticatedUsers which returned only one event. To confirm this was the right one I inspected the event and found:

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/a73dc95c6e2269e79170a59fbbefbb5759c53063/Evidence/SPL_Q4_EVIDENCE/Q4_PublicAcess_Evidence.png)

This showed that the group, AllUsers was granted permission for READ/WRITE.

### Q5 – What is Bud’s username?
![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/a73dc95c6e2269e79170a59fbbefbb5759c53063/Evidence/SPL_Q5_EVIDENCE/Q5_Username_Evidence.png)

I looked back into where I inspected the eventID for the public PutBucketAcl and scrolled down to the bottom of the event info and noted that it displayed the userIdentity.type attribute of IAMUser which links back to Q1 where I had to list the IAM users. 

This led me to find that the user who made the bucket public was bstoll.

### Q6 – What is the name of the bucket that was made publicly accessible?
I needed to find the name of the bucket that was made public by bstoll. This again was simple as the answer was within the evidence I have provided for Q4 and Q5

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/a73dc95c6e2269e79170a59fbbefbb5759c53063/Evidence/SPL_Q6_EVIDENCE/Q6_BucketName_Evidence.png)

By reviewing the CloudTrail PutBucketAcl event that made the bucket accessible, I can see that there is one attribute which stands out, bucketName: frothlywebcode, which is the answer I am looking for.

### Q7 – What is the name of the text file that was successfully uploaded into the S3 bucket while it was publicly accessible?
![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/a73dc95c6e2269e79170a59fbbefbb5759c53063/Evidence/SPL_Q7_EVIDENCE/Searching%20for%20any%20uploads%20to%20bucket.png)

I used aws:s3:accesslogs to pull the server access logs from the dataset. 

I then filtered it to search for the bucket name (frothlywebcode) we previously found in Q6 alongside http_method=”PUT” and http_status=200 so that it was filtered to successful upload operations which made it much easier to find the necessary logs needed.

To make it easily readable, I outputted it in a table where I could see the time of any uploads alongside the key which was would provide me with the .txt file I needed, which was:

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/a73dc95c6e2269e79170a59fbbefbb5759c53063/Evidence/SPL_Q7_EVIDENCE/Q7_FileName%2BExtension_Evidence.png)

### Q8 – What is the FQDN of the endpoint that is running a different Windows operating system edition than the others?
![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/a73dc95c6e2269e79170a59fbbefbb5759c53063/Evidence/SPL_Q8_EVIDENCE/Searching%20for%20OS%20field.png)
![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/a73dc95c6e2269e79170a59fbbefbb5759c53063/Evidence/SPL_Q8_EVIDENCE/Searching%20for%20OS%20field%202.png)

By running a check to find all the OS related fields within the dataset to help find fields containing information about the OS edtions which led to me finding a field that stated exactly that:

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/0091d25805c275c990d11a416c2bca34cdfec620/Evidence/SPL_Q8_EVIDENCE/OS%20editions.png)

Following this, I then compared the OS values by host to output a list of each host that was running the OS versions to help me find which host was making use of a different OS version compared to others.

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/0091d25805c275c990d11a416c2bca34cdfec620/Evidence/SPL_Q8_EVIDENCE/Searching%20for%20odd%20(OS)%20one%20out.png)

The output I received made it clear which was the anomaly out of all the hosts, however that was only the start. I still had to find the FQDN to get my full answer (BSTOLL-L.*****.**)

I then pivoted to filtering my search to events regarding BSTOLL-L.

![altt text](https://github.com/Chris-Bratosin/COMP3010/blob/0091d25805c275c990d11a416c2bca34cdfec620/Evidence/SPL_Q8_EVIDENCE/Searching%20for%20BSTOLL-L%20events%20(found%20potential%20FQDN).png)

This pivot returned a small set of events under the syslog sourcetype with the host splunkhwf.froth.ly.

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/0091d25805c275c990d11a416c2bca34cdfec620/Evidence/SPL_Q8_EVIDENCE/Found%20the%20FQDN%20in%20event%20payload.png)

When I reviewed the event payload for syslog events, the FQDN was visible in the VSN field which confirmed that the endpoint’s FQDN was BSTOLL-L.froth.ly.

Q6 also was able to provide some supporting context for this as earlier CloudTrail evidence gave some reference to the froth.ly domain which was consistent with FQDN that was in the vsn field.

![alt text](https://github.com/Chris-Bratosin/COMP3010/blob/8a54e4401d50876a65e3de2feb7286ce6fd9755f/Evidence/SPL_Q8_EVIDENCE/Supporting%20Evidence%20from%20Q6.png)
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
