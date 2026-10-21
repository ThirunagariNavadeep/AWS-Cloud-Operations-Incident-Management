# Incident Report: Controlled HTTP 500 Test

## Summary

A controlled HTTP 500 response was generated to validate Apache access logging, the CloudWatch Logs metric filter, the custom `HTTP5xxCount` metric, and the CloudWatch alarm/SNS notification path.

## Environment

- EC2 instance: `aws-ops-lab`
- Operating system: Amazon Linux 2023
- Web server: Apache HTTP Server (`httpd`)
- AWS Region: `ap-southeast-2` (Sydney)
- Log group: `/aws/ec2/aws-ops-lab/apache-access`
- Metric: `AWSOpsLab/Apache` → `HTTP5xxCount`
- Alarm: `aws-ops-lab-http-5xx-errors`
- SNS topic: `aws-ops-lab-alerts`

## Evidence observed

- Apache was active.
- A real request to `/ops-lab-test-500` returned HTTP status `500`.
- Apache recorded the request in `access_log` with status `500`.
- A `HTTP5xxCount` datapoint with value `1` was visible in CloudWatch.
- Alarm history showed a transition from `OK` to `In alarm` and a successful SNS action.
- An email notification was received during testing.

**Correlation note:** The alarm was also manually forced into the ALARM state during testing. Therefore, the alarm history and SNS action prove the notification path, but the real-event causal link should be claimed only after correlating the real HTTP 500 datapoint timestamp with the alarm evaluation and state transition.

## Cleanup

The temporary Apache test configuration was intended to be removed after validation. Verify that `/etc/httpd/conf.d/ops-lab-test-error.conf` no longer exists and that `sudo apachectl configtest` returns `Syntax OK`.

## Follow-up actions

- Verify the alarm's missing-data treatment and threshold.
- Keep SSH restricted to a trusted IP.
- Capture screenshots of the log event, metric datapoint, alarm history, and SNS email with sensitive account details obscured.
- Stop the EC2 instance when it is not needed.
