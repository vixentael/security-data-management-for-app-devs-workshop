**Workbook** for "Security data management for app devs" workshop during [try! Swift world](https://www.tryswift.co/world/#). By Anastasiia [@vixentael](https://twitter.com/vixentael).



# Table of contents
1. [Defining data model, risks, threats, security controls](#datamodel)
    1. [Example](#datamodel-example)
    2. [Exercises](#datamodel-exercises)
2. [Software security controls](#controls)
3. [Security tools](#tools)
4. [Regulations](#regulations)
5. [Security design guidelines](#design-guidelines)
6. [Risk management guides](#risk-mngm-guides)
7. [Security verification and testing](#testing)
8. [Resources](#resources)

---



<a name="datamodel"></a>

## Defining data model, risks, threats, security controls.

<a name="datamodel-example"></a>

#### **Example** 

Imagine Note-taking app or Todo app. Users want to put very secret notes there and be sure that notes are protected from app developers and Apple. Developers implemented notes encryption, that leads to the following data model. App uses iCloud backend and doesn't handle user login/authorisation, as it goes through iCloudKit.

##### Example 1.1. Data model of Note-taking app.

| **Code** | **Data type**       | **Description**                                              |
| :------- | :------------------ | :----------------------------------------------------------- |
| **NT**   | plaintext note      | Generated by user, synced across user devices via iCloud. Users can create many **NTs**. |
| **PW**   | user password       | Generated by user input or iOS Keychain, password required to generate **NEk**. One **PW** per app. |
| **NEk**  | note encryption key | Generated by app from **PW**, used to "lock and encrypt", then "unlock and decrypt" confident notes. One **NEk** per **NT**. |

Different risks can have different impact on organization/users. Higher risk means more harm.



##### Example 1.2. Risks to data of Note-taking app. 

Levels: critical, high, moderate, low.

| Risks   | Access                                                       | Disclosure                                                   | Modification                                                 | Access denial                                                |
| :------ | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **NT**  | Moderate *(note is inevitably* *displayed on the device* *screen at some point)* | Critical *(this is exactly what users want to be protected from)* | Critical                                                     | High *(losing note text —* *making users angry)*             |
| **PW**  | Moderate *(having password alone* *won’t help to decrypt notes)* | Critical *(users tend to reuse* *passwords, we should* *avoid having them* *in plaintext)* | Critical *(can’t decrypt* *notes linked to* *this password)* | Critical *(losing password —* *losing all encrypted notes)*  |
| **NEk** | Moderate *(used for encryption* *of one note)*               | Low *(used for encryption* *of one note)*                    | Low *(wrong key — decryption* *of a specific note* *is impossible)* | Moderate *(lost key — decryption* *of a specific note* *is impossible)* |

Risks are theoretical until attackers find the way to how exploit them. Attack vectors are such ways.



##### Example 1.3. High priority attack vectors.

High-priority attack vectors (how attackers can implement risks): 
- data transmission between app and application backend (broken transport encryption or broken **NT** encryption).
- vulnerabilities in 3rd party libraries (f.e. vulnerability in analytics SDK which leads to sharing **NT** with 3rd party).
- unidentified (so far) iCloud vulnerabilities.
- iOS jailbreak attacks (including remote JB) that leads to stealing **NT** or **PW.**
- iCloud Keychain vulnerabilites (leads to stealing **PW**)



##### Example 1.4. Security controls against high risks and high priority attack vectors.

From threats and risks became obvious, that app shouldn't use **PW** in plaintext, as **PW** is critical resource and can be stolen due to iOS-specific vulnerabilties.

| **Data class** | **Security control  (transfer)**                             | **Security control  (storage)**                              |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **NT**         | TLS.<br/>Note encryption.<br/>iCloud certificate pinning.    | Stored encrypted.<br/>**NEk** is unique per each note and is not saved in persistent storage (calculated in memory).<br/>Auto-locking timer (after 1 min of inactivity **NT** gets encrypted).<br/>**NT** is never appears in plaintext persistently, only stored in memory variable while on screen. |
| **PW**         | Not transferred by app,<br/>might be transferred iCloudKeychain. | Stored as hash using password-based KDF and app-generated salt.<br/>Stored as hash in Keychain/iCloudKeychain.<br/>Keychain has additional biometrics protection (if enabled by user). |
| **NEk**        | Not transferred.                                             | Not stored persistently, calculated before usage.<br/>Memory variable is filled with zeros after usage.<br/>Removed from memory by auto-locking timer. |

##### Example 1.5. Read more.

Real-world public example of risk managing for Note-taking app is Bear app. I've described building end-to-end encryption for Bear app in [this blogpost](https://www.cossacklabs.com/blog/end-to-end-encryption-in-bear-app.html). 



---

<a name="datamodel-exercises"></a>

#### Exercises.

Imagine app that allows users to see their sensitive documents and share them to each other (aka "SecureDropbox"). App has own backend. Based on information about app's data suggests risks model, attack vectors and security controls.



##### Exercise 1.1. Review data model of Document app.

| **Code** | **Data type**                   | **Description**                                              |
| :------- | ------------------------------- | ------------------------------------------------------------ |
| **D-US** | User session info               | JWT token, email, password that are in the app Keychain while the user is logged in. Email might be considered as PII according to GDPR, CCPA and others. |
| **D-AK** | API keys of 3rd party libraries | API tokens of libraries that app uses. Part of app bundle, stored in plist. |
| **D-IC** | Integrity checksums             | Hashes of some components (3rd party library index.html file) that are stored as part of app bundle. Used to verify that 3rd party library was not modified. |
| **D-BI** | User business info              | Name, title, company, phone number. Not stored persistently, received using API call, cached in memory while app is running. Regulated by GDPR, CCPA. |
| **D-DC** | Documents                       | User has access to documents (pdf, docx, xls files) of particular projects. Users don’t generate documents from the app, just read them. Regulated by GDPR, CCPA. |



##### Exercise 1.2. Fill in empty spaces in risks to data of Document app.

| Risks    | Access | Disclosure | Modification | Access denial |
| :------- | :----- | :--------- | :----------- | :------------ |
| **D-US** |        | High       |              |               |
| **D-AK** |        |            | Low          |               |
| **D-IC** | Low    |            |              | High          |
| **D-BI** |        |            |              |               |
| **D-DC** | High   | Critical   | Critical     |               |



##### Exercise 1.3. Define high-priority attack vectors for Documents app.

High-priority attack vectors: 

- data transmission between app and application backend (broken transport encryption).
- privileges escalation of user role (access to documents/projects that user shouldn’t have).
- .... (continue the list)
- ....



##### Exercise 1.4. Define security controls against high risks and high priority attack vectors for Documents app.

Based on what you learnt about risks and attack vectors, suggest protection measures:

| **Data class** | **Security control  (transfer)**                             | **Security control  (storage)**                              |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **D-US**       | TLS.<br/>Successful user authentication.<br/>Client certificate pinning. |                                                              |
| **D-AK**       |                                                              | Stored encrypted by static in-app key, gets decrypted on app running. |
| **D-IC**       | Not transferred.                                             |                                                              |
| **D-BI**       |                                                              | Not stored persistently.                                     |
| **D-DC**       | Successful user authentication.                              | Not stored persistently.<br/>Removed from cache when user closes document.<br/>Auto-locking timer. |



<a name="controls"></a>

## Software security controls for mobile apps

There're many security controls that you can use to improve mobile app security — some of them are software-based for your app, some make sense on your app backend, some are organization-based (security awareness trainings, password reset policies, etc), some rely on 3rd party service providers (security engineering, security design, pen testing, security code audits, etc). 

Noone knows the full list of security controls (but [NIST SP 800-53](https://csrc.nist.gov/publications/detail/sp/800-53/rev-4/final) has good ideas), this is a short list tailored for mobile devs. Also, [OWASP MASVS](https://github.com/OWASP/owasp-masvs) checklist is a good start.



**Transport/network:**

* proper TLS settings
* certificate pinning (client certificate pinning)
* additional encryption layer, application level payload encryption (like, end-to-end encryption of data of the same user, or encryption of data from userA to userB)
* app doesn't rely on a single insecure communication channel for critical operations

**Storage:**

* data encryption
* keys encryption
* keys management
* no storing data outside of app container/system secret storage
* storing in Keychain/SecureEnclave
* storing in Keychain with biometrics-protection
* wiping data when it became not needed
* "secure wipe out" of data with "zeroing" (fill variable with zeroes before nulling)
* backup of important user data (suggest to backup) 

**Access control:**

* device pin protection
* biometrics authentication
* user session authentication (OAuth2, JWT, cookies, etc)
* user session expiration/termination
* authenticating user before performing critical operation (removing project, changing password)
* roles managing (ABAC/RBAC)

**Anti-reverse engineering and JB-protection:**

* app checks if device is jailbroken, especially on performing critical actions, and alerts users
* app detects when run on simulator
* app detects when run using debugging tools
* anti-bruteforce timers/counters to limit amount of attempts
* code obfuscation
* integrity checks of loaded libraries (when library is added by package manager, when app calls library in runtime)

**Platform**

* app is signed with valid credentials, noone has access to account's provisioning profiles and private keys
* debugging symbols are removed in release app
* app only depends on up-to-date connectivity and security libraries
* app does not export sensitive functionality via custom URL schemes

**Monitoring:**

* security testing/monitoring of 3rd party dependencies
* honeypots (fake user credentials, API requests, pieces of code)
* tracking "risky" user behaviour (unsuccessful login attempts, opening app on JB device, decryption errors)
* security events monitoring systems (SIEMs)



<a name="tools"></a>

## List of (defensive) appsec tools for mobile apps

Noone know full list of useful security tools for mobile apps. Check services like [tools.tldr.run](https://tools.tldr.run/).

**Multi-platform high-level encryption libraries:**

* [Themis](https://github.com/cossacklabs/themis)
* [SwiftSodium](https://github.com/jedisct1/swift-sodium)
* [RNCryptor](https://github.com/RNCryptor/RNCryptor)

**Apple-first encryption libraries:**

* CryptoKit (high level)
* CommonCrypto (low level)

**Secrets management:**

- [awslasb/git-secrets](awslasb/git-secrets) — prevents from committing secrets
- [cocoapods-keys](https://github.com/orta/cocoapods-keys) by @orta — keep secrets away from github
- [gitleaks](https://github.com/zricethezav/gitleaks) — scans repo for missing secrets
- [ios-datasec-basics](https://github.com/vixentael/ios-datasec-basics) — repo that describes methods how to encrypt API keys before storing in the app

**Dependency checks (SCA, software composition analysis):**

* [WhiteSourceSoftware](https://www.whitesourcesoftware.com/) plugin for iOS, known to check Carthage
* [BlackDuckSoftware](https://www.blackducksoftware.com/) pluging for iOS, known to check CocoaPods
* [Snyk](https://snyk.io/test/) checks for React Native and Cordova
* npm audit, github notification, python, [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/) — for backend code

**Obfuscation, code hardening, threat protection :**

* [dexprotector](https://dexprotector.com/), [tutorial for iOS](https://dev.to/shostarsson/application-obfuscation-on-ios-3d2c)
* [GuardSquare](https://www.guardsquare.com/en)
* [Swift Shield](https://github.com/rockbruno/swiftshield)
* [blogpost with other tools](https://www.polidea.com/blog/open-source-code-obfuscation-tool-for-protecting-ios-apps/)

**Anti-JB protection (see above, plus):**

* [disable debug](https://gist.github.com/mattlawer/d391c5137f987d109c54b58a7ce36a04)
* [detect debugging](https://gist.github.com/julepka/de2c9094118d47112e22dc7761579e3b)
* [jailbreak detection blogpost](https://duo.com/blog/jailbreak-detector-detector)

**SAST/DAST**

* [checkmarx](https://www.checkmarx.com/)
* [sonarqube](https://www.sonarqube.org/)
* [veracode](https://www.veracode.com/products/binary-static-analysis-sast)
* [appknox](https://www.appknox.com/dynamic-application-security-testing)
* [app-ray](https://app-ray.co/)
* [blogpost with other tools](https://www.softwaretestinghelp.com/mobile-app-security-testing-tools/)

Beware: automated tools typically require ongoing attention due to large number of false positives / false negatives.


<a name="regulations"></a>

## Data protection regulations

Regulations differ for country/region, industry, business activity. 

The list is not limited to:

**General data security compliance:** ISO/IEC 27002:2013, CCPA, NIST 800-171, FIPS 140-2, GDPR, DPA, Brazilian General Data Protection Act, etc

**Finance**: PCI DSS, PCI HSM, SWIFT Customer Security Controls, PSD2, FINMA, GLBA, etc

**Healthcare**: HIPAA, HITECH, ISO 27799:2016, etc

**Education**: FERPA, etc

[Apple Export regulations on cryptography](https://docs.cossacklabs.com/pages/apple-export-regulations/)



[**Cheatsheet** on major regulations](https://www.cossacklabs.com/blog/what-we-need-to-encrypt-cheatsheet.html), what data they require to protect.



<a name="design-guidelines"></a>

## Mobile application security design (standards and guidelines)

[Apple platform security guides](https://support.apple.com/guide/security/welcome/web) — 157p pdf, describes Apple's hardware security, system security, encryption and data protection, app security, network security and dev kits. Useful to understand environment app's running in, platform restrictions and security controls that Apple use themselves.

[iOS security guidelines](https://github.com/0xmachos/iOS-Security-Guides) — iOS-specific guidelines that were updated till iOS 12.3, then merged with Apple security guides (see above).

[Privacy chapter from App Store review guidelines](https://developer.apple.com/app-store/review/guidelines/#privacy) — describes App Store guidelines on privacy policy, data collection, storage, use, and sharing.

[NIST SP 800-53, Security and Privacy Controls for Federal Information Systems and Organizations](https://csrc.nist.gov/publications/detail/sp/800-53/rev-4/final) — describes numerous technology-independent security controls, classifies them by "family", usage, functionality, capabilities, etc. 



<a name="risk-mngm-guides"></a>

## Risk management guides

*Mobile world doesn't have separate risk management guidelines, just another types of risks. Check software risk management frameworks and apply to mobile dev according to mobile app risks.* 

[OWASP mobile risks top10](https://owasp.org/www-project-mobile-top-10/) — short list of most popular risks for mobile apps made by OWASP, dated 2016. Useful for quick understanding of typical risks and their mitigations.

[FAIR risk assessment](https://www.fairinstitute.org/) — rather short (comparing to others) framework on risk calculation and triage.

[NIST SP 800-37 "Risk Management Framework for Information Systems and Organizations: A System Life Cycle Approach for Security and Privacy"](https://csrc.nist.gov/publications/detail/sp/800-37/rev-2/final) — describes risk management of application development inside organization, provides clear guidance for large organizations. Use it if you work in large organization and need to establish risk management process across deparments, or read to feel how simple mobile app development is in comparison.



<a name="testing"></a>

## Mobile application security verification and testing

[OWASP Mobile application security verification standard (MASVS)](https://github.com/OWASP/owasp-masvs) — checklist of mobile app security requirements, different depending on app/data risks, business requirements and regulations. Use it to measure "security scores" for your app — how good it is — and to track progress by repeating MASVS checks every 6-12 months.

[OWASP Mobile security testing guide (MSTG)](https://github.com/OWASP/owasp-mstg) — describes *how* to perform security verifications from MASVS, gives ideas and pieces of code. Use it as "tutorial" to MASVS.

[NIST SP 800-163, Vetting the Security of Mobile Applications](https://csrc.nist.gov/publications/detail/sp/800-163/rev-1/final) — mobile app vetting is a process to ensure that mobile app conforms to an organization’s security requirements and is "reasonably free" from vulnerabilities. Describes vetting process (including budgeting and staff) and typical iOS/Android app vulnerabilities.

[OWASP Software assurance maturity model (SAMM)](https://owaspsamm.org/) — "checklist" for the whole organization that produces software, covers not only software properties, but organization dev process (understanding threats, selecting security providers, taking care about software security architecture). Use it to measure how good software security is in your organization.



<a name="resources"></a>

## More links

**iOS-specific security things, tips how to start with your app security**

- ["Popular note-taking apps share these security flaws: security tips for developers"](https://medium.com/@vixentael/popular-note-taking-apps-share-these-security-flaws-security-tips-for-developers-326180e41329) 
- [X Things you Need to Know before Implementing Cryptography](https://speakerdeck.com/vixentael/x-things-you-need-to-know-before-implementing-cryptography) by [@vixentael](https://twitter.com/vixentael)
- [By-passing biometrics protection](https://speakerdeck.com/julep/making-authentication-more-secure) slides by [@julepka](https://twitter.com/julepka)
- [End-to-end encryption for Wire](http://www.swifttube.co/video/end-to-end-encryption-for-ios-developer-mihail-gerasimenko) by @GerasimenkoMiha
- [Building end-to-end encryption for Bear app](https://speakerdeck.com/vixentael/10-lines-of-encryption-1500-lines-of-key-management) by @vixentael

**Books**

* [NoStarch iOS security book](https://nostarch.com/iossecurity)
* [Password authentication for web and mobile apps](https://dchest.com/authbook/) book by Dmitry Chestnykh
* [API security](https://developer.okta.com/books/api-security/)

**Online courses / workshops**

- [online course on reverse engineering iOS apps](https://github.com/ivRodriguezCA/RE-iOS-Apps) by [@ivRodriguezCA](https://twitter.com/ivRodriguezCA)
- [other talks and workshops on iOS app security](https://github.com/vixentael/my-talks) by [@vixentael](https://twitter.com/vixentael)



## Stay tuned

We do data security for living. At Cossack Labs we help companies who process sensitive data and want to protect it against external attackers, insiders, mis-configurations and to be compliant with regulations. We build and license our own proprietary or open source software, or build custom solutions tailored for specific use cases.

Ping [@vixentael](https://twitter.com/vixentael) or [@cossacklabs](https://cossacklabs.com/), if you need security engineering assistance :)
