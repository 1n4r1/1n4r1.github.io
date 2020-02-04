---
layout: post
title: Summary - MITRE ATT&CK
categories: Security-framework
---

![placeholder](https://inar1.github.io/public/images/2020-02-04/mitre_attack_matrix.png)

# Explanation
To dig into the enterprise level information security knowledge (Meaning not a CTF or something like that !!),<br>
summarize the security framework "MITRE ATT&CK".

# Last Update
2020/02/04

# Solution
## 1. Who published it?
The <a href="https://www.mitre.org/">MITRE Corporation</a> is an American not-for-profit organization based in Bedford, Massachusetts, and McLean, Virginia.


## 2. When it was published?
Started 2013 and updated every quarter.


## 3. Why it is important?
1. There used to be no common frameworks for consulting, penetration test, SOC that is for both attack side and defence side.
2. The ATT&CK framework has become widespread rapidly in the security product industry since approximately 2018.


## 4. What is the "MITRE ATT&CK" ?
"<a href="https://attack.mitre.org/">MITRE ATT&CK</a>" is a cybersecurity framework which stands for "Adversarial Tactics, Techniques, and Common Knowledge".<br>
They use matrix for the describing each attack method. The followings are the matrix they have<br>
What we have to care is "Enterprise" is mostly for the post-exploitation while Pre-ATT&CK is for the recon, weaponization and delivery.
1. Enterprise Matrix
2. Mobile Matrix
3. Pre-ATT&CK Matrix

The following <a href="https://attack.mitre.org/resources/enterprise-introduction/">model</a> explains what "PRE-ATT&CK" and "ATT&CK" cover.
![placeholder](https://inar1.github.io/public/images/2020-02-04/enterprise-pre-lifecycle.png)

To explain, this time I use the "<a href="https://attack.mitre.org/matrices/enterprise/">Enterprise Matrix</a>".<br>
The following is the part of Enterprise Matrix.
![placeholder](https://inar1.github.io/public/images/2020-02-04/2020-02-01-16-07-53.png)

We have 3 keyword for this matrix.
1. <a href="https://attack.mitre.org/tactics/enterprise/">Tactics</a> : The header of the matrix. Separated as sub categories.
2. <a href="https://attack.mitre.org/techniques/enterprise/">Techniques</a> : The body of the matrix. Attack techniques / Attack vectors for the each Tactics.
3. <a href="https://attack.mitre.org/mitigations/enterprise/">Mitigations</a> : Mitigation for the attack methods.

Then, go to each Techniques.<br>
This time, the example is "<a href="https://attack.mitre.org/techniques/T1106/">Execution through API</a>.
![placeholder](https://inar1.github.io/public/images/2020-02-04/2020-02-03-21-04-24.png)

Each Techniques has the following information.
1. General description
2. Procedure examples : What Adversary groups / tools have leveraged this Technique.
3. Mitigations : How to mitigate the risk of this Technique.
4. Detection : How to detect this Technique.
5. References


## 5. What we can do with "ATT&CK" ?
On the document "<a href="https://www.mitre.org/sites/default/files/publications/pr-18-0944-11-mitre-attack-design-and-philosophy.pdf">MITRE ATT&CK™: Design and Philosophy</a>", we have the following use case of ATT&CK.
### 1. Adversary Enumeration
> The process of assessing the security of a technology domain by applying cyber threat intelligence about specific adversaries and how they operate to emulate that threat.

### 2. Red Teaming
> Applying an adversarial mindset without use of known threat intelligence for the purpose of conducting an exercise.

### 3. Behavioral Analytics Development
>  behavioral detection analytics can be used to identify potentially malicious activity within a system or network that may not rely on prior knowledge of adversary tools and indicators. 

### 4. Defensive Gap Assessment
> A defensive gap assessment allows an organization to determine what parts of its enterprise lack defenses and/or visibility. 

### 5. Soc Maturity Assessment
> ATT&CK can be used as one measurement to determine how effective a SOC is at detecting, analyzing, and responding to intrusions.

### 6. Cyber Threat Intelligence Enrichment
> Cyber threat intelligence covers knowledge of cyber threats and threat actor groups that impact cybersecurity.


## 6. Tools / Resources
### Tools are the simulators of testing "ATT&CK" implementation.
1. <a href="https://github.com/uber-common/metta">Uber Metta</a>
Tutorial: <a href="https://medium.com/uber-security-privacy/uber-security-metta-open-source-a8a49613b4a">https://medium.com/uber-security-privacy/uber-security-metta-open-source-a8a49613b4a</a>

2. <a href="https://github.com/mitre/caldera">MITRE Caldera</a>
Website: <a href="https://www.mitre.org/research/technology-transfer/open-source-software/caldera"></a>
Video tutorial: <a href="https://www.mitre.org/research/technology-transfer/open-source-software/caldera"></a>

3. <a href="https://github.com/redcanaryco/atomic-red-team">Red Canary atomic Red Team</a>
Website: <a href="https://redcanary.com/atomic-red-team/">https://redcanary.com/atomic-red-team/</a>

4. <a href="https://github.com/endgameinc/RTA">Endgame Red Team Automation</a>
Website: <a href="https://www.endgame.com/blog/technical-blog/introducing-endgame-red-team-automation">https://www.endgame.com/blog/technical-blog/introducing-endgame-red-team-automation</a>

### Documents of log expected result of testing "ATT&CK" implementation.
1. <a href="https://car.mitre.org/">MITRE Cyber Analytics Repository</a>
Knowledge base of analytics based on the MITRE ATT&CK adversary model.

2. <a href="https://pan-unit42.github.io/playbook_viewer/">Palo Alto Unit 42 Playbook Viewer</a>
Free playbook based on ATT&CK by Palo Alto Unit 42.

3. <a href="https://static1.squarespace.com/static/552092d5e4b0661088167e5c/t/5b8f091c0ebbe8644d3a886c/1536100639356/Windows+ATT%26CK_Logging+Cheat+Sheet_ver_Sept_2018.pdf">Malware Archaeology Windows ATT&CK Logging Cheat Sheet</a>
Document explains what happens when Windows server logs being attacked.

### Other tools / Documents
1. <a href="https://mitre-attack.github.io/attack-navigator/">ATT&CK navigator</a>


## 7. FAQ
* <a href="https://attack.mitre.org/resources/getting-started/">Frequently Asked Questions</a>


## 8. Conclusion

Please someone give me a correction if there is an incorrect description or lacking information!!
