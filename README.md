# wafUriTriager

When DDoS occurs, many organizations are not equipped with the tools to mitigate the attack quickly. After managed and rate-based rules, many customers are at a loss on how to further block offenders without simultaneously breaking connectivity for legitimate users.

The WAF URI Triager is a project to help organizations quickly identify and mitigate DDoS which is too complex to be automatically mitigated by managed rule groups or rate-based rules, or for those organizations with specific restrictions preventing them from using those features. 

Some key points:

- The architecture diagram includes an WAF-protected ALB, but none of the functional components are reliant on ALB or the EC2 target group. This architecture will work for any WAF-protected resource on AWS.

- Many of the features of this architecture are highly customizable. While the patterns described in the diagram outline a use case for identifying only 3 items block-able items from malicious requests: Country of Origin, User Agent, and the targeted URI path; these fields can be customized or even automated to make extremely complex WAF rules. With many fields to choose from, users can customize how complex CloudWatch Logs Insights become and create much more complex CLI-formatted WebACL create-rule API calls. If an organization wanted to instead integrate their own query solution or machine learning, this becomes much more intelligent.

How does it work?

- At a high level, the WAF URI Triager seeks to quickly identify attack patterns used by a DDoS actor which cannot quickly be aggregated by IP. As many bots and IPs will be used, the commonality will likely come down to a few key features of the request including, but not limited to, destination URI, origin country, and user agent. By creating a complex rule using a combination of these parameters, you can quickly mitigate malicious actors while allowing legitimate users to continue to access your application. The more dynamic your legitimate user-base is, the more complex you should consider making these queries and your rules.

- On a schedule via EventBridge, this architecture will provide a daily rollup to security teams within your organization so that you can have an active monitor of commonly probed locations on your web app. The same queries will be triggered for the daily rollup as will for the alarm-based rollup.

- At a threshold you determine, a CloudWatch Alarm will be triggered based on a CloudWatch Logs Metric Filter which can be configured to identify re-occurrences of rate-based rules triggering. The conditions for this filter can be customized as well. When this alarm triggers, Lambda will perform the same topOffenders and topHits queries and output immediately to S3, which will trigger a secondary Lambda to format the identified attack patterns into a new rule. The reason for providing the command to you via email vs. automating is to ensure that it has been signed off on prior to potentially interrupting any legitimate traffic. One could modify this automatically deploy rules, but be warned of potential interruptions to legitimate traffic.

- Once the ALARM state has resolved, the Lambda will go back to its normal query patterns based on the EventBridge schedule.

- The S3 bucket can be set with your desired retention period. You may find it useful to keep data for longer periods including the daily rollups.
