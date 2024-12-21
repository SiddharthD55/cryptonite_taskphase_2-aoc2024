This focuses on uncovering the activities of an unauthorized user who gained access to a CCTV server, manipulated footage, and deleted critical records. Using Splunk as the primary tool for log analysis, I was able to identify the attacker’s entry point, their actions, and the methods they employed to cover their tracks.

Through custom field extraction, advanced SPL queries, and detailed correlation of logs across multiple sources, I pieced together the attack timeline and provided actionable insights for mitigating similar incidents in the future.

The investigation was guided by the following goals:

1. **Parse Unstructured Logs**: Extract custom fields from improperly parsed logs to make the data usable for analysis.
2. **Identify Suspicious Activity**: Detect rare or anomalous events that deviate from normal patterns.
3. **Trace the Attacker**: Correlate logs from the CCTV application and the web server to identify the source of unauthorized access.
4. **Establish a Timeline**: Build a comprehensive timeline of events leading up to, during, and following the attack.


## Tools and Technologies

- **Splunk**: For indexing and querying log data.
- **Regex**: To create custom field extractions from unstructured logs.
- **Visualization Tools**: Pie and bar charts for summarizing data patterns.
- **SPL (Search Processing Language)**: For querying and transforming log data.
- **Audit Trails**: Cross-referenced logs from the CCTV server and web server for correlation.


## Methodology

### 1. **Initial Exploration of Logs**

I began by reviewing the ingested logs in Splunk to understand their structure and content:

```spl
index=* sourcetype=cctv_logs
```

It became clear that many fields were not automatically parsed, leaving critical data embedded in unstructured text. For example, fields like `Event`, `UserName`, and `Session_id` were missing, which hindered my ability to analyze user activities effectively.


### 2. **Field Extraction and Parsing**

To make the data usable, I manually extracted fields using Splunk's "Extract Fields" feature. This involved selecting key portions of the log text and generating a regex pattern. The initial regex looked like this:

```regex
^(?P<timestamp>\d+-\d+-\d+\s+\d+:\d+:\d+)\s+(?P<Event>(Login\s\w+|\w+))\s+(?P<user_id>\d+)?\s?(?P<UserName>\w+)\s+.*?(?P<Session_id>\w+)$
```

#### Challenges Encountered
- **Diverse Log Formats**: Some logs (e.g., failed login attempts) had extra details that broke the parsing logic.
- **Field Misalignment**: Certain fields, like `Session_id`, appeared inconsistently across events.

#### Solution
To address these issues, I revised the regex for better coverage:

```regex
^(?P<timestamp>\d+-\d+-\d+\s+\d+:\d+:\d+)\s+(?P<Event>[^\s]+)\s+(?P<UserName>[^\s]+)\s+.*?(?P<Session_id>[^\s]+)?$
```

This allowed me to extract essential fields regardless of log format inconsistencies.

### 3. **Analyzing User Activities**

With fields properly extracted, I moved on to analyzing user activities. My focus was on identifying patterns of normal behavior and deviations that could indicate malicious intent.

#### Event Counts by User
I queried the data to count the number of events per user:

```spl
index=cctv_logs | stats count(Event) by UserName
```

This revealed which users were most active, providing a baseline for identifying anomalies.

#### Rare and Suspicious Events
Next, I searched for rare events that might be linked to the attack, such as failed login attempts or deletion commands:

```spl
index=cctv_logs | rare Event
```

This query highlighted a series of unusual actions, including:
- **Multiple Failed Login Attempts**: A clear indicator of brute-force activity.
- **Delete Commands**: A rare event that aligned with the deletion of CCTV footage.

### 4. **Correlating Logs Across Sources**

To identify the attacker, I correlated logs from the CCTV server with web server logs. My hypothesis was that the attacker’s IP address or session ID would appear in both sources, providing a direct link.

#### Searching Web Logs for Session IDs
Using a session ID from the CCTV logs (`rij5uu4gt204q0d3eb7jj86okt`), I queried the web server logs:

```spl
index=web_logs *rij5uu4gt204q0d3eb7jj86okt*
```

This revealed the attacker’s IP address: **10.11.105.33**.

#### Tracing Activities by IP
I used the attacker’s IP to isolate all related activities across both log sources:

```spl
index=* 10.11.105.33
```

This confirmed that the IP was linked to multiple failed logins, a successful login, and the deletion of CCTV footage.


### 5. **Constructing the Attack Timeline**

By piecing together the logs, I reconstructed the sequence of events:

1. **Brute-Force Login Attempts**: The attacker repeatedly attempted to log in using various usernames.
2. **Successful Login**: Eventually, they gained access using valid credentials.
3. **CCTV Footage Accessed**: Logs showed the attacker watching and downloading camera streams.
4. **Footage Deletion**: Finally, the attacker deleted key footage, likely to conceal their actions.

## Findings and Recommendations

### Key Findings
- The attacker exploited weak account credentials to gain access.
- Logs provided a clear trail of the attacker’s activities, from brute-forcing to deletion.
- Correlating logs from multiple sources was essential to identifying the perpetrator.

### Recommendations
1. **Enhance Authentication**: Implement multi-factor authentication (MFA) and enforce strong password policies.
2. **Improve Log Hygiene**: Ensure all logs are consistently formatted and include detailed metadata.
3. **Set Up Real-Time Alerts**: Use Splunk alerts to flag suspicious patterns, such as multiple failed logins or rare events like deletions.
4. **Conduct Security Audits**: Regularly review user accounts, system configurations, and access logs for vulnerabilities.
5. 
## So...

This demonstrates the importance of detailed log analysis in uncovering and understanding security incidents. By leveraging Splunk’s capabilities and employing a systematic approach, I successfully traced the attacker, identified their methods, and proposed actionable steps to prevent future breaches.

**Answers**

*Extract all the events from the cctv_feed logs. How many logs were captured associated with the successful login?* **642**
 
*What is the Session_id associated with the attacker who deleted the recording?* **rij5uu4gt204q0d3eb7jj86okt**
 
*What is the name of the attacker found in the logs, who deleted the CCTV footage?* **mmalware**
