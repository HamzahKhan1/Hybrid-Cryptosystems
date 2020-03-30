# **Hybrid-Cryptosystems**

## **Background**
The world of cryptography runs insanely deep, so I'll be providing a short but limited background here. Most security related jobs don't require extremely specific knowledge of cryptography, but understanding it's context and necessity is imperative.
- **Cryptography** is often defined as the study of techniques for secure communication, which involves creating codes that allow info to remain secret. It's goal is to ensure Privacy, Authentication, Integrity, and Non-repudiation (P.A.I.N. framework).
- **Encryption** is the process of encoding a message or other information is such a way that only authorized parties can see it. In a more technical sense, it involves converting plaintext into ciphertext with the use of an encryption algorithm and some sort of key.
- **Symmetric key algorithms** use the same key to encrypt and decrypt info, and are less computationally intense. They use Advanced Encrytion Standard, or AES.
- **Asymmetric key algorithms** use two keys, a public and private key. They are more computationally intense, but easier to use with many people, and involve the verification of one's identity via digital signatures. They use Rivest-Shamir-Adleman standard, or RSA.
- **Hybrid cryptosystems**, which we'll be using today, combine the convenience of a asymmetric systems with the efficiency of symmetric systems. 

## **Objective**
- Use a hybrid cryptosystem to exchange encrypted messages with another party

## **Setup**
This has a wide range of applications, but we'll go ahead and use a Linux distribution for our encryption process. Most of our work will be done on the command line interface.
- It's recommended to have a partner for this project, but if you don't have one, you can easily complete it on your own by sending files and messages between two VMs. If you choose to do this, it's recommended to use a service such as Slack (you can just boot it up on the web browser) to transfer files painlessly. I used the **Ubuntu** and **Kali** distributions just fine to run this.

## **Process** 
1. First, **make a new directory, change into it, and create a file with your own unique message**. This can be done with `echo '[message]' >> encryptionfile.txt` (in my case, the file is named dirty_little_secret - if you want to simplify things for yourself, go ahead and follow the example in the picture).

![image](https://user-images.githubusercontent.com/55573209/77864203-ee276080-71ec-11ea-8743-2b6d2a362804.png)

2. Next, we'll be using OpenSSL to **generate an RSA keypair**. If you remember, this covers the asymmetric half of the hybrid system. 

Create a 2048 bit RSA private/public keypair with:

`openssl genrsa -des3 -out private.pem 2048`

Extract the public key from this keypair with:

`openssl rsa -in private.pem -outform PEM -pubout -out public.pem`

![image](https://user-images.githubusercontent.com/55573209/77864205-f1bae780-71ec-11ea-9271-fc18ba0fc874.png)

- See the picture below, taken from the OpenSSL man page, to understand what OpenSSL is and how it functions:

![image](https://user-images.githubusercontent.com/55573209/77864261-275fd080-71ed-11ea-99ef-f7ad9203ef73.png)

3. **Send yourself or your partner your public key** via Slack, and have them do the same (yes, if you're doing the project on your own, this means you will have to follow the same steps, but on a different VM). 

- Make sure to rename the other person's public key to "partners_public.pem". If you don't you risk duplicating and overwriting your own public key. 

![image](https://user-images.githubusercontent.com/55573209/77864207-f4b5d800-71ec-11ea-89e6-55980c93642b.png)

4. Now, we'll be handling the symmetric side of things by using OpenSSL to **generate a symmetric key and Initialization Vector**, which is a unique number that AES needs to encrypt your data securely.

Run `openssl enc -aes-256-cbc -nosalt -k password -P | tee secrets` and then cat the 'secrets' file. You'll see your symmetric key and IV codes. 

- Create a file called symmetrickey.dat. Copy and paste your key from secrets into symmetrickey.dat. Do the same thing with the IV into a file called iv.dat.

![image](https://user-images.githubusercontent.com/55573209/77864209-f8495f00-71ec-11ea-824b-cdbc48f515e8.png)

5. Now it's time to use the symmetric key to **encrypt messages**. Run `openssl enc -nosalt -aes-256-cbc -in dirty_little_secret.txt -out dirty_little_secret.enc -base64 -K <key> -iv <IV>`.

- In place of <key> and <IV>, copy and paste the key from your symmetrickey.dat and iv.dat files, respectively.
  
- You'll now have an encrypted version of your message in dirty_little_secret.enc - you can double check by using the `cat` command.

![image](https://user-images.githubusercontent.com/55573209/77864219-01d2c700-71ed-11ea-939c-c3fea794d83a.png)

6. Now it's time to use your partner (or other VM's) public key to **encrypt your symmetric key**. They will then use their private key to decrypt the symetric key, which can then be used to decrypt your message (or vice-versa). 

To encrypt your symmetric key, run:

`openssl pkeyutl -encrypt -in symmetrickey.dat -inkey partners_public.pem -pubin -out symmetrickey.enc`

![image](https://user-images.githubusercontent.com/55573209/77864221-06977b00-71ed-11ea-9f4b-c68d57c00285.png)

7. Now, it's time to **trade a couple of files** between you and your partner (or yourself). 

- Open up Slack and send symmetrickey.dat.enc, iv.dat, and dirty_little_secret.enc to your partner, and have them send them to you too.

- When you both get the files, download them from Slack. They should show up either in your downloads folder or your home directory. Pictures for both included below.

- Before you move them into your HybridCryptosystems directory, make sure to ename them to partners_dirty_little_secret.enc, partners_symmetric_key.enc, and partners_iv.dat in order to avoid confusion. 

![image](https://user-images.githubusercontent.com/55573209/77864232-11521000-71ed-11ea-9e07-177cab13770b.png)

![image](https://user-images.githubusercontent.com/55573209/77864237-157e2d80-71ed-11ea-8b3b-2f6112aae136.png)

8. Time for the final, and most satisfying step - **decryption**. 

- First, you'll be using your partner's public key to decrypt their symmetric key. Then, you'll use that decrypted key to decrypt their secret message.

To decrypt the symmetric key, use:

`openssl pkeyutl -decrypt -in partners_symmetric_key.enc -inkey private.pem -out partners_symmetric_key.pem`

To decrypt the secret message with symmetric key, run:

`openssl enc -aes-256-cbc -d -nosalt -in partners_dirty_little_secret.enc -base64 -K <partner's symmetric key> -iv <partner's IV>`

- Note: Copy and paste your partner's symmetric key where the <partner's symmetric key> placeholder is, and your partner's IV where <partner's IV> appears.

![image](https://user-images.githubusercontent.com/55573209/77864250-1b740e80-71ed-11ea-9892-e3802ba42de5.png)

## **Results**
- You'll now, finally, be able to read you or your partner's once encrypted message. 
- As you can see in the picture above, I was finally able to read the message that I wrote in Kali after decrypting it in Ubuntu (and provided I follow the steps correctly, I would be able to do the same fom Ubuntu --> Kali). 

## **Notes** 
- Keep in mind that this kind of encryption happens millions (or more) times a day whenever you are browsing the internet. The point of this project is to understand the process and the concepts involved. 
- In security, although there are always tools to make a process easier (Nmap can make your ping sweep script feel redundant, for example) it's incredibly important to have a strong foundational understanding of how a process works so that you can communicate potential errors and issues to others, even if your technical knowledge doesn't directly transfer over to your own work. 
