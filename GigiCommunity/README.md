# Gigi Project Publication System Design

## Overview

This document provides a detailed conceptual design for a system that facilitates the publication of Gigi projects through a moderated Distributed Hash Table (DHT). The system is designed to ensure that projects are securely broadcasted, reviewed, and published to the community in a controlled and secure manner.

## Workflow

<img src="workflow_diagram.png" alt="Workflow Diagram">

### 1. Broadcasting a Gigi File
When a user decides to broadcast a Gigi file to the community, the file is first encrypted using asymmetric encryption. This ensures that the file can only be decrypted by the intended recipient, who holds the corresponding private key. The encrypted file is then sent to DHT1, which serves as a temporary storage location for files awaiting approval.

### 2. Approval Process
Once the encrypted file is stored in DHT1, a designated decryption key holder retrieves and decrypts the file for review. The decryption key holder is responsible for evaluating the content of the file to ensure it meets the community's standards and guidelines. If the file is approved, the decryption key holder re-encrypts the file using a secure re-encryption scheme and submits it to DHT2. Along with the re-encrypted file, a revocation key is also generated and stored, allowing the decryption key holder to revoke the file if necessary.

### 3. Publication
The re-encrypted file is now stored in DHT2, which serves as the permanent storage location for approved Gigi files. The encryption scheme used for DHT2 ensures that the file is unencryptable by the general community, providing an additional layer of security. However, the entry in DHT2 can be revoked by the decryption key holder of DHT1 using the revocation key, allowing for the removal of the file if it is later deemed inappropriate or harmful.

### 4. User Access to DHT2
Community members can access DHT2 to retrieve and decrypt approved Gigi files. Each file in DHT2 is re-encrypted to a shared community key that authorized community members possess. Users can use this shared key to decrypt and access the files. The system ensures that only approved content is accessible, maintaining the integrity and security of the community's resources.

## Components

<img src="component_diagram.png" alt="Component Diagram">

### DHT1
- **Purpose:** DHT1 acts as a temporary storage location for encrypted Gigi files that are awaiting approval. It ensures that files are securely stored and only accessible to the designated decryption key holder.
- **Encryption:** Asymmetric encryption is used to protect the files in DHT1, ensuring that only the decryption key holder can access and decrypt the files for review.
- **Schema:**
  - `file_id`: Unique identifier for the file.
  - `encrypted_file`: The encrypted Gigi file.
  - `timestamp`: The time when the file was uploaded.
  - `uploader_id`: Identifier of the user who uploaded the file.
  - `status`: Current status of the file (e.g., pending, approved, revoked).
- **Multi-party OPRF:** Integrating Multi-party Oblivious Pseudorandom Function (OPRF) can enhance the security and privacy of the approval process. By allowing multiple parties to jointly compute a pseudorandom function without revealing their inputs, it ensures that sensitive information remains confidential during the review and approval stages. This can prevent any single party from having complete knowledge of the file's content, thereby reducing the risk of unauthorized access or tampering. Additionally, Multi-party OPRF can be configured to require two approvers for a publication, further enhancing the security and integrity of the approval process.

### DHT2
- **Purpose:** DHT2 serves as the permanent storage location for approved Gigi files. It ensures that only approved files are accessible to the community while providing a mechanism for revocation if necessary.
- **Encryption:** Files in DHT2 are re-encrypted using a secure re-encryption scheme that makes them unencryptable by the general community. This ensures that only the intended recipients can access the files, while still allowing for revocation by the decryption key holder of DHT1. Additionally, files are re-encrypted to a shared community key that everyone in the community has, allowing authorized community members to decrypt and access the files if they possess the shared key.
- **Schema:**
  - `file_id`: Unique identifier for the file.
  - `re_encrypted_file`: The re-encrypted Gigi file.
  - `timestamp`: The time when the file was approved and stored.
  - `approver_id`: Identifier of the decryption key holder who approved the file.
  - `revocation_key`: Key used to revoke the file if necessary.

## Reputational Star-Based System

<img src="reputation_diagram.png" alt="Reputation Diagram">

To encourage quality contributions and maintain a high standard within the community, a reputational star-based system is implemented. Users can rate files they access from DHT2 based on their quality and usefulness. Each file can receive a rating from 1 to 5 stars, and these ratings are aggregated to provide an overall score for the file.

- **User Ratings:** After accessing and reviewing a file, users can submit a rating. This rating is stored along with the file's metadata in DHT2.
- **Reputation Score:** The aggregated star ratings contribute to the file's reputation score, which is visible to all community members. Higher scores indicate higher quality and more useful contributions.
- **Incentives:** Users who consistently contribute high-quality files and receive high ratings can earn badges or other forms of recognition within the community. This incentivizes users to maintain high standards and contribute valuable content.

## Security Considerations

<img src="security_diagram.png" alt="Security Diagram">

- **Asymmetric Encryption:** The use of asymmetric encryption ensures that files are securely transmitted and stored in DHT1. Only the designated decryption key holder can access and decrypt the files for review.
- **Re-Encryption Schemes:** The system leverages advanced re-encryption schemes such as BBS98 (Blaze-Bleumer-Strauss) and ElGamal-based re-encryption. BBS98 is secure under the assumption of the hardness of the Decisional Diffie-Hellman (DDH) problem on elliptic curves, while ElGamal-based re-encryption relies on the Elliptic Curve Discrete Logarithm Problem (ECDLP), which is very secure with large key sizes (e.g., 256-bit curves like secp256r1).
- **Conditional Proxy Re-Encryption:** The Ateniese-Gaeta-Nita scheme adds conditions to the re-encryption process, providing fine-grained control and enhanced security for scenarios requiring conditional access.
- **Multiparty Oblivious Pseudorandom Function (OPRF):** Integrating Multiparty OPRF can enhance the security and privacy of the approval process. By allowing multiple parties to jointly compute a pseudorandom function without revealing their inputs, it ensures that sensitive information remains confidential during the review and approval stages. This can prevent any single party from having complete knowledge of the file's content, thereby reducing the risk of unauthorized access or tampering. Additionally, Multiparty OPRF can be configured to require two approvers for a publication, further enhancing the security and integrity of the approval process.
- **Resistance to Known Attacks:** Modern re-encryption schemes that have been rigorously scrutinized by the cryptographic community and have resisted known attacks are employed to ensure high security.
- **Use of Modern Elliptic Curves:** The system uses well-regarded elliptic curves like secp256r1, secp384r1, or Curve25519 with secure key management practices to enhance security. Older curves or those with known weaknesses are avoided.

## Threat Modeling and Mitigations

### Threats
1. **Unauthorized Access to DHT1:**
   - **Threat:** An attacker gains unauthorized access to DHT1 and attempts to decrypt files.
   - **Mitigation:** Use of asymmetric encryption ensures that only the designated decryption key holder can decrypt the files. Regular audits and access controls can further mitigate this risk.

2. **Compromise of Decryption Key Holder:**
   - **Threat:** The decryption key holder's private key is compromised, allowing an attacker to decrypt and approve malicious files.
   - **Mitigation:** Implement multi-factor authentication and secure key storage practices. Regularly rotate keys and use Multi-party OPRF to require multiple approvers for added security.

3. **Tampering with Files in Transit:**
   - **Threat:** An attacker intercepts and modifies files during transmission between users and DHT1 or between DHT1 and DHT2.
   - **Mitigation:** Use of end-to-end encryption and secure communication protocols (e.g., TLS) to protect files in transit. Implement integrity checks to detect any tampering.

4. **Revocation Key Misuse:**
   - **Threat:** An attacker gains access to the revocation key and revokes legitimate files.
   - **Mitigation:** Store revocation keys securely and limit access to authorized personnel only. Implement logging and monitoring to detect any unauthorized revocation attempts.

5. **Denial of Service (DoS) Attacks:**
   - **Threat:** An attacker overwhelms the system with requests, causing a denial of service.
   - **Mitigation:** Implement rate limiting, load balancing, and other DoS mitigation techniques to ensure system availability.

6. **Insider Threats:**
   - **Threat:** A malicious insider with access to the system attempts to bypass security measures or leak sensitive information.
   - **Mitigation:** Implement strict access controls, regular audits, and monitoring to detect and prevent insider threats. Use Multi-party OPRF to ensure no single party has complete control over the approval process.

## Future Work

- **Implementation of Advanced Re-Encryption Schemes:** Develop and integrate robust re-encryption schemes such as BBS98, ElGamal-based re-encryption, and Ateniese-Gaeta-Nita to securely transfer files from DHT1 to DHT2. This will involve researching and selecting appropriate re-encryption algorithms and implementing them within the system.
- **User Interface:** Design and develop a user-friendly interface that allows users to easily broadcast, review, and publish Gigi files. The interface should provide clear instructions and feedback to users throughout the process.
- **Moderation Tools:** Create a set of tools to assist decryption key holders in the review and approval process. These tools should include features for decrypting, reviewing, re-encrypting, and submitting files, as well as managing revocation keys.
- **Integration of Multiparty OPRF:** Explore the integration of Multiparty OPRF to further enhance the security and privacy of the approval process. This will involve researching the best practices for implementing Multiparty OPRF and ensuring it aligns with the overall system design.

## Conclusion

This design document provides a comprehensive overview of the Gigi project publication system. By leveraging asymmetric encryption, advanced re-encryption schemes, Multiparty OPRF, and a moderated DHT, we aim to create a secure and efficient workflow for broadcasting, reviewing, and publishing Gigi projects to the community. The system ensures that only approved content is accessible to the community while providing mechanisms for revocation and content moderation. The addition of a reputational star-based system further encourages quality contributions and helps maintain high standards within the community.

