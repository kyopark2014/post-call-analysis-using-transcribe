# LLM-post-call
It shows how to manage post call analysis.

## Transcribe를 이용한 변환

```python
{
    "TranscriptionJobName": "TestCallCenter",
    "LanguageCode": "en-US",
    "MediaSampleRateHertz": 8000,
    "MediaFormat": "wav",
    "Media": {
        "MediaFileUri": "s3://filesharing-ksdyb/post-call/sample-call-1.wav"
    }
}
```

[boto3 - start_transcription_job](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/transcribe/client/start_transcription_job.html#)

[boto3 - start_job](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/amplify/client/start_job.html#)

[boto3 - start_call_analytics_job](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/transcribe/client/start_call_analytics_job.html)
