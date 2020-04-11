# **Incident-Response-Traffic-Analysis-Exercise**

## **Background**
Today's project is a walkthrough through the incident response process from the eyes of a junior SOC (Security Operations Centers) analyst who's been asked to determine if an incident is a false positive or a true incident. 

## **Objective**
We'll be using the "Timbershade" exercise files from [malware-traffic-analysis.net](https://malware-traffic-analysis.net/2019/01/28/index.html), a super handy site full of training exercises to analyze pcap files of malicious network traffic. 

### **Key Questions**
During our analysis, we'll attempt to answer a couple of key questions that will determine if the incident is a true threat or not:

1. What activity is being reported, and at what date and time was it reported at?
2. What IP addresses (external and internal) and source/destination ports are being flagged for malicious activity?
3. What are the MAC addresses of the computers involved, and what's the host name of the internal machine involved?
4. Can you confirm if the alert is accurate, if the malware was downloaded, and whether the alert is a false positive or true positive?
5. What steps can be taken to mitigate the risk?
6. What kind of attack is this (Web/Email/Network)?

## **Tools**
A **virtual machine** (VM) is a key tool in the kit of a security analyst. They provide you with a safe place to analyze harmful malware, foreign links, and other suspicious files. While they aren't totally impervious to infections, they serve as an excellent first layer of defense.
- We'll be using the **Ubuntu Linux** VM, and inside of it, we'll be working on **terminal** and **Wireshark**, a packet capture software.

## **Process**

1. On your VM, download the .txt and pcap files from [here](https://malware-traffic-analysis.net/2019/01/28/index.html). Open a Terminal window, and use the command `unzip` followed by the names of the files. Then, use the password `infected` to unlock them. 

![1](https://user-images.githubusercontent.com/55573209/79034471-9dfcb680-7b7b-11ea-8025-43b67de12c19.png)

2. Use the `cat` command to view the .txt snort alert file. Comb through it and notice the unique alert headlines, such as these:

![2](https://user-images.githubusercontent.com/55573209/79034475-a81eb500-7b7b-11ea-8eaa-30f616e4057e.png)

![3](https://user-images.githubusercontent.com/55573209/79034478-ace36900-7b7b-11ea-9218-d840f32301d8.png)

3. We can infer that some kind of malicious file has been downloaded off the internet, considering the "HTTP request" and potentially hostile "file download" alerts. Take note of the IP addresses and source and destination ports listed in the second alert, which pertains to the file download (external IP address: 91.121.30.169, internal IP address: 172.17.8.109, source port: 8000, destination port: 49207).

4. Now, it's time to pivot into Wireshark to get some more information. We want to find out more about that HTTP request, so open up the pcap file and use the dynamic filter `http.request and ip.addr == 172.17.8.109 and ip.addr == 91.121.30.169` to narrow our search results down to the IP addresses we found in the alert file. You'll now see only one packet, looking like this:

![4](https://user-images.githubusercontent.com/55573209/79034481-bbca1b80-7b7b-11ea-9777-161f9ec1e53d.png)

5. Notice a few key things: the timestamp, the source and destination IP's, the protocol, and the 'Info' row, which describes a GET request of some sort of file. 

6. Also notice in the "Packet Details" box that we can now see the MAC address of the computers involved in this incident, next to `Ethernet II, src:` (pictured above).

7. Now, right click on the packet and select `Follow HTTP Stream` to see something like this: 

![5](https://user-images.githubusercontent.com/55573209/79034485-ce445500-7b7b-11ea-9a55-50705c9eada7.png)

8. We see the GET request again, and want to find out more about this mysterious `/91msE95B/actiV.bin` file. Let's get it onto our machine so we can analyze it further. Go to `File > Export Objects > HTTP` and you’ll get a new screen with two files. click `Save` to get them on your file system (remember: do this on your VM, *not* on your host machine).

![6](https://user-images.githubusercontent.com/55573209/79034719-22e8cf80-7b7e-11ea-934b-fe46dd59c7fb.png)

9. Perfect! Before we take a look at these two files, let's confirm the name of the host computer of the internal machine we saw earlier. Still in Wireshark, filter for DHCP traffic using `bootp` (bootstrap protocol, the protocol used by DHCP). Go to the bottom pane, expand the “Bootstrap Protocol (Inform)” option, and look for the `Host Name` option. Expand it, and we find the answer to our question: **Dunn-Windows-PC**. 

![7](https://user-images.githubusercontent.com/55573209/79034870-d6ea5a80-7b7e-11ea-97a6-633b6bd9835c.png)

10. Now, we want to examine those two suspicious files in the Terminal. Use `cat` to take a look, and you'll see this:

![8](https://user-images.githubusercontent.com/55573209/79034883-e23d8600-7b7e-11ea-99ae-296ad4ce46d4.png)

11. Examining the `ncsi.txt` file gives us two words - ‘Microsoft NCSI’. This doesn’t give us much info, but luckily, something more interesting is going on with the `actiV.bin`, which seems to be encrypted.

12. Instead of further tampering with this potentially malicious file, we’ll follow a safe process to get more info on it by getting it’s hash. Run `md5sum actiV.bin`, and you’ll generate a unique hash like this:

![9](https://user-images.githubusercontent.com/55573209/79034976-89222200-7b7f-11ea-9d42-33d8e4c276b2.png)

13. Now, open your firefox browser and visit `virustotal.com`.  This is a handy browser tool that acts as a free virus, malware and URL scanner that analyzes suspicious files to detect potential threats and then automatically shares them with the security community. Go to `Search` and copy paste the unique hash you just generated in the terminal.

![10](https://user-images.githubusercontent.com/55573209/79035000-be2e7480-7b7f-11ea-9460-9a6ac80ae5db.png)

14. `actiV.bin` is as suspicious as it looks; the file tests positive for 59 malicious malwares. 

![11](https://user-images.githubusercontent.com/55573209/79035012-ddc59d00-7b7f-11ea-944a-b8229e20d970.png)

![12](https://user-images.githubusercontent.com/55573209/79035014-e322e780-7b7f-11ea-8191-904ded120622.png)

15. The `details`, `relation`, `behavior`, and `community` tabs will give you a lot of fascinating insight into the file, but for now, we have the information we need. We now know the snort alert was accurate, considering the `behavior` page shows us the HTTP requests and malicious files, and we can categorize the issue as a True Positive Web attack, since it involved a victim clicking on a malicious web link and activating an HTTP GET request to grab a malicious file. 

16. By this point, your job as a junior analyst might be done - however, fterwards, to mitigate the risk, you could write a Wireshark rule for the malicious IP with the following process: 

![13](https://user-images.githubusercontent.com/55573209/79035063-647a7a00-7b80-11ea-937c-916b26361afd.png)

- #IPv4 source address

`iptables --append INPUT --in-interface etho --source 91.121.30.169/32 --jump DROP`

17. At this point, the threat has been thoroughly examined, identified,  determined to be a true positive, and even mitigated!

## **Conclusion**

If you are relatively new to the realm of security like I am at the time of writing this, then this project can be an exciting culmination of a lot of the skills you've spent the last couple of weeks or months learning. 

In this activity alone, you employ knowledge of the command line, networking protocols, cryptography and encryption, Wireshark, VM's, firewalls/network security, and IDS' in order to make the right call on a potential incident!
