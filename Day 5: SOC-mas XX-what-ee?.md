Well here I understood the importance of incorporating robust security practices during development, particularly for applications with significant societal impact like Wareville's WishVille platform.

The exploration revealed a **critical XML External Entity (XXE) vulnerability** in the wishlist feature of the platform. By exploiting the vulnerability, it was possible to:
1. Access sensitive server files (e.g., `/etc/hosts`).
2. Bypass administrative restrictions to view user-submitted wishes stored as files in `/var/www/html/wishes/`.

#### **Root Cause**
The vulnerability originated from insecure XML processing:
- The use of `libxml_disable_entity_loader(false)` enabled loading external entities.
- Insufficient sanitization or validation of XML payloads allowed malicious input.

### **Exploitation Steps**
1. **Initial Discovery**: 
   - Captured the XML payload for the "Add to Wishlist" feature using Burp Suite.
   - Identified the presence of user-controlled XML parsing logic.
   
2. **Payload Construction**: 
   - Modified the XML payload to include a malicious external entity:
     ```xml
     <!DOCTYPE foo [<!ENTITY payload SYSTEM "/etc/hosts"> ]>
     <wishlist>
       <user_id>1</user_id>
       <item>
         <product_id>&payload;</product_id>
       </item>
     </wishlist>
     ```
   - Sent the payload, successfully retrieving the `/etc/hosts` file.

3. **Sensitive Data Access**: 
   - Guessed file paths (`/var/www/html/wishes/`) based on typical web server structures.
   - Iterated through files (e.g., `wish_1.txt`, `wish_2.txt`) to access user-submitted wishes.

### **Impact Assessment**
- **Data Breach**: Sensitive user data, including private wishes, was exposed.
- **Administrative Control Bypass**: The XXE attack circumvented access controls.
- **Potential for Further Exploitation**: The vulnerability could allow attackers to:
  - Extract additional sensitive server files (e.g., `/etc/passwd`).
  - Perform server-side request forgery (SSRF) to interact with internal network services.

### **Mitigation Recommendations**
1. **Disable External Entity Loading**:
   - Use `libxml_disable_entity_loader(true)` or equivalent in the XML parser.
   - Ensure the `LIBXML_NOENT` flag is not used unless absolutely necessary.

2. **Input Validation and Sanitization**:
   - Reject XML input containing potentially harmful keywords or unexpected structures.
   - Enforce strict schemas (e.g., via Document Type Definitions, DTDs) to limit allowed elements.

3. **Secure Development Practices**:
   - Conduct thorough security testing as part of the CI/CD pipeline.
   - Incorporate static and dynamic analysis tools to detect vulnerabilities early.

4. **Access Controls**:
   - Implement proper authentication and authorization mechanisms to restrict access to sensitive files.

### **Conclusion**
The incident underscores the risks of prioritizing deadlines over security, particularly in applications that handle sensitive user data. The oversight in skipping security testing resulted in a critical vulnerability that could have jeopardized the townspeople's trust in the WishVille platform.

Through the efforts of McSkidy and the team, the vulnerability was identified and remediated, ensuring the safety of the platform for Wareville's citizens. The developers were reminded of the importance of secure coding practices, and steps were taken to integrate security into the development lifecycle moving forward. As for the Mayor, further investigation may be warranted to clarify their role in the introduction of the vulnerability.

**Answers**

*What is the flag discovered after navigating through the wishes?* **THM{Brut3f0rc1n6_mY_w4y}**

*What is the flag seen on the possible proof of sabotage?* **THM{m4y0r_m4lw4r3_b4ckd00rs}**
