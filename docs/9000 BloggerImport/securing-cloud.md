---
title: 'Securing the Cloud'
date: 2016-09-04T13:27:00.001+01:00
draft: false
aliases: [ "/2016/09/securing-cloud.html" ]
parent: Blogger
---
#### date: 2016-09-04

With all the breaches recently one does wonder how we can protect ourselves better.

The cloud service provider (CSP's) talk about encryption like it's a silver bullet, cloud HSM for example, but is it really?

After all, ye who has physical control, has total control of an asset.

We have three states that data can be at during the use of encryption.

1) At rest

2) In transit

3) In Process

Now the Cloud Security Alliance advise not to keep your keys in the cloud and they should reside on premise, this makes absolute sense to me and would be my preferred choice, the cloud provider should have zero knowledge right!

So for data 'at rest' we can store the keys on our devices and during 'in process' the key is still on our device converting the data into encrypted data, we can then write the encrypted data to cloud storage. nothing in clear text hits the CSP and they have zero knowledge and cannot decrypt our data.

For data 'in transit' I accept communications from third parties and negotiate secure protocols and ciphers to use for the session. The message is in clear text between third party and my device however we have secured the transmission of this message from snooping parties, I can then choose to store this message and apply 'at rest' encryption. So far were all good and pretty confident that no one else can access our data, even though we are using cloud storage.

Now for data 'in process' we have to be very careful. Now so far the processing is taking place on our own assets which we have physical control over and can be physically secured in locked rooms with relevant access controls etc etc.

Why do we have to be very careful?

Well the 'in process' state is the most vulnerable point for your data. You have to store your private keys and decrypted data in memory in order to process it. So if you choose to outsource this state to the cloud then complete assurance cannot be determined and only best effort is applied.

Let me explain how I've come to this conclusion. As stated previously, private keys and decrypted data reside in memory. A virtual machines ram is not encrypted, even if it was at the hypervisor level, the CSP would have the key. The CSP could take a snapshot and then create a ram dump to extract the private keys. The CSP would also know what other servers have been communicating to/from and could deduce where storage might be in use and attempt using the recovered key with such storage, given the limited information exposed by the CSP from the logs you may never know you're under attack.

Now all your secrets are exposed, but in reality how could this be possible.

Let's assume that a determined attacker infiltrates an organisation posing as a cloud architect/engineer, perhaps their role is just the mole and another person infiltrates again as a cloud engineer, they all have clean records so pass all the vetting, they can now begin to target whomever is of interest, perhaps work up the supply chain and into the target. Your completely reliant upon the CSP attempts to detect these rogue employees.

The other scenario is a government issues a subpoena and gag order and the CSP becomes complicit in exposing all your secrets, I feel this scenario is more likely given recent whistle blowing.

So is that cloud HSM still appealing now?

I think even if you had the keys in one CSP and the data in another you'd have to pick the jurisdiction wisely. I know I don't have any information that is likely to keep any nation any safer, but I'd still like to keep the information I do have private.

If you've the opinion that you have nothing to hide, then I urge you to send your local spooks all your accounts and passwords and let me know the outcome.

So what can we do about this issue with data 'in process' in the cloud?

Well for a start you would have to use IaaS as you have the most control over this.

I did explore the idea of encrypting the Ram, and there is a linux operating system patch that does such a thing called Tresor see;

https://en.wikipedia.org/wiki/TRESOR

https://www.usenix.org/legacy/event/sec11/tech/full\_papers/Muller.pdf (2011)

The conclusion was that a VM was still not safe because it emulates the registers and a soft reboot would maintain the last state in the x86\_debug register, however the recent (2010) CPU extension for AES-NI were more promising but at the time popular hypervisor's did not expose this to the VM.

I have to research this further as to whether it's now working, but the CSP will need to support AES-NI in the API/GUI and allow policy to ensure the VM only migrates to hypervisor with this functionality.

Even if ram encryption became defacto we'd just be moving the problem down to the CPU and JTAG ports reading the registers, assuming they are not emulated.

So ultimately nothing can be done about this issue as it stands today. So ultimately it boils down to 'do you trust your CSP' and 'Are you happy that Governments are reading your data' if it's yes then outsource, otherwise keep this function to your private cloud.

other articles of interest using search terms 'vm ram encryption'

http://www.blackkettle.org/blog/2015/02/19/youve-got-to-trust-your-vm-host/

https://www.virtualizationpractice.com/safe-way-to-encrypt-within-a-vm-need-for-technology-6336/

http://www.computerworld.com/article/2475269/security0/prism-proof-solution-for-the-public-cloud--security-salvation-or-snake-oil-.html