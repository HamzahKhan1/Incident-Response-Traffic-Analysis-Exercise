# **Incident-Response-Traffic-Analysis-Exercise**

## **Background**
- Today's project is a walkthrough through the incident response process from the eyes of a junior SOC (Security Operations Centers) analyst who's been asked to determine if an incident is a false positive or a true incident. 

## **Objective**
- We'll be using the "Timbershade" exercise files from [malware-traffic-analysis.net](https://malware-traffic-analysis.net/2019/01/28/index.html), a super handy site full of training exercises to analyze pcap files of malicious network traffic. 

### **Key Questions**
During our analysis, we'll attempt to answer a couple of key questions that will determine if the incident is a true threat or not:

1. What activity is being reported, and at what date and time was it reported at?
2. What IP addresses (external and internal) and source/destination ports are being flagged for malicious activity?
3. What are the MAC addresses of the computers involved, and what's the host name of the internal machine involved?
4. Can you confirm if the alert is accurate, if the malware was downloaded, and whether the alert is a false positive or true positive?
5. What steps can be taken to mitigate the risk?
6. What kind of attack is this (Web/Email/Network)?

## **Tools**
- A **virtual machine** (VM) is a key tool in the kit of a security analyst. They provide you with a safe place to analyze harmful malware, foreign links, and other suspicious files. While they aren't totally impervious to infections, they serve as an excellent first layer of defense.
- We'll be using the **Ubuntu Linux** VM, and inside of it, we'll be working on **terminal** and **Wireshark**, a packet capture software.

## **Process**

1. On your VM, download the .txt and pcap files from [here](https://malware-traffic-analysis.net/2019/01/28/index.html). Open a Terminal window, and use the command `unzip` followed by the names of the files. Then, use the password `infected` to unlock them. 




## **Notes**
