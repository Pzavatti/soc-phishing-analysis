### SOC Level 1 Incident Report – Phishing Email Analysis

> **NOTE:** This analysis is strictly for educational and training purposes.
> No organizational systems or production environments were involved.
> I am self-teaching cybersecurity and this is my first SOC analysis
> project, part of a growing security portfolio.

## What I Learned

- How to read and interpret raw email headers (SPF, DMARC, ARC)
- How to identify display name spoofing vs. actual sender domain
- Recognising spam filter evasion techniques (text poisoning,
  dictionary stuffing, encoded strings)
- How attackers abuse trusted infrastructure (Google Cloud Storage)
  to bypass URL filters
- How to defang IOCs for safe public sharing
- How to document and structure a SOC-style threat analysis

## Incident Summary

A suspicious email impersonating Costco was received in a personal mailbox and analyzed for phishing indicators as part of a SOC Level 1 training exercise. The email attempted to lure me (the recipient) into engaging with a fraudulent promotional offer (“YETI Beach Lounge Wagon”) using urgency and exclusivity-based social engineering techniques.
The sender was identified as `eVfRECLH[@]uklveaejx[.]us`, which is not associated with any legitimate Costco domain (e.g., Costco.com or Costco.ca). The display name was spoofed as “Costco,” indicating brand impersonation.

## Initial Analysis

A basic overview of the email shows several key artifacts indicating malicious intent. The subject line of the email simply copies my personal email address and concatenates the headline: Your Costco YETI Beach Lounge Wagon Awaits; the sender does not make the effort to personalize it by using my name. The email claims to be from Costco, however, the sending address is: `eVfRECLH[@]uklveaejx[.]us`, which appears to be randomly generated and is not associated with any legitimate Costco domain such as “Costco.ca”. Further, next to the sending address we find the following string: `via 74152158[.]220[.]112[.]58qqvd2nbsbc9g8g158[.]220[.]112[.]5gmytspl32mupe2y[.]musicuplift[.]com`; a highly obfuscated return-path string incorporating IP fragments and randomized alphanumeric sequences, which suggests automated phishing kit generation and potential relay abuse through compromised or disposable domains. The body of the email presents urgency-based social engineering, a fake promotional reward offer, and a call-to-action directing the user to an external URL hosted on Google Cloud storage (storage.googleapis.com). While Google infrastructure is legitimate, it is commonly abused to host phishing landing pages due to its trusted reputation and open hosting capabilities. Finally, the “Unsubscribe” button is also part of the scam as it leads to the same malicious cloud storage page as the main link.

## Email Authentication Analysis

Further investigation in the source code of the email reveals conclusive evidence of its malicious nature. The email authentication results indicate multiple security failures: SPF PASS (for the attacker’s domain, not Costco); DMARC: FAIL (No alignment with “from:” domain); DKIM: MISSING (No cryptographic signature present). Although SPF passed, it only validated the attacker infrastructure and not the legitimate brand domain. The DMARC failure confirms misalignment between the visible sender identity and authenticated sending domain, indicating likely spoofing or unauthorized email sending. Additionally, the absence of DKIM further strengthens the phishing classification.

## Header and Infrastructure Analysis

The email header provides us with further evidence of the malicious intent presenting several anomalies. The email originates from IP address `158[.]220[.]112[.]5`, hosted on a Contabo server infrastructure `(vmi3127950[.]contaboserver[.]net)`, which is commonly associated with VPS hosting environments frequently abused in spam and phishing campaigns. The message ID `(654445494-654445494[@]654445494[.]com)` appears to be artificially generated, and the timestamp discrepancy (converted to 31 December 1969) suggests a manipulated header data, often associated with automated spam tools or improperly configured mail systems. These patterns are consistent with phishing infrastructure designed to evade detection and track user engagement.

## The Hash Busting

Below the email’s body, in the source code, we can observe a bizarre chunk of unrelated text about lemon trees, how to take care of plants, a bunch of unrelated words, ASCII art, what seems to be pieces of old emails to other recipients, and blocks of randomly generated hexadecimal strings of text. It’s gibberish added to confuse spam filters so they don't recognize the email as a repetitive scam template, this technique is known as “Hash Busting”.

## Conclusion

Overall, this email demonstrates a phishing attempt leveraging brand impersonation, infrastructure abuse, authentication misalignment, and content obfuscation techniques. The objective appears to be either credential harvesting or user engagement with a malicious external landing page under the premise of a Costco promotional reward. The combination of SPF PASS on a non-legitimate domain, DMARC FAIL, missing DKIM, suspicious hosting infrastructure, and heavily obfuscated content results in a high-confidence malicious classification.

## Recommended Actions:

    • Block sender domain: `uklveaejx[.]us`
    • Block sending IP: `158[.]220[.]112[.]5`
    • Block associated redirect URLs (Google Cloud Storage path)
    • Conduct IOC sweep for similar email patterns
    • Report to phishing intelligence feeds if applicable

## Final thoughts

The IP `158[.]220[.]112[.]5` belongs to Contabo, a VPS provider. Attackers often rotate through dozens of these. Blocking one is good, but stay alert for others in the same subnet.
The domain `uklveaejx[.]us` was likely registered for a low price just for this campaign. It will probably never be used again once it's flagged.
It would help to push a “Security Awareness Notice” to all the departments who were possibly affected by the phishing attempt.
