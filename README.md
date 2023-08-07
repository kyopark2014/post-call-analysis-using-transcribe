# LLM을 위한 post-call 분석

## Transcribe의 Post-call Analytics를 이용한 변환

### Post-call 분석 요청 

API Request는 아래와 같습니다. 

```java
{
    "CallAnalyticsJobName": "myPostCallTest",
    "Media": {
        "MediaFileUri": "s3://filesharing-ksdyb/post-call/sample-call-1.wav"
    },
    "Settings": {
        "LanguageOptions": [
            "en-US"
        ]
    },
    "ChannelDefinitions": [
        {
            "ChannelId": 1,
            "ParticipantRole": "AGENT"
        },
        {
            "ChannelId": 0,
            "ParticipantRole": "CUSTOMER"
        }
    ],
    "DataAccessRoleArn": "arn:aws:iam::677146750822:role/service-role/AmazonTranscribeServiceRoleFullAccess-TrascribeRoleForPostCall"
}
```

이것을 구현하는 코드는 아래와 같습니다.

```python
import boto3

CallAnalyticsJobName = "myPostCallTest01"
MediaFileUri = "s3://filesharing-ksdyb/post-call/sample-call-1.wav"
LanguageOptions = "en-US"
DataAccessRoleArn = "arn:aws:iam::677146750822:role/service-role/AmazonTranscribeServiceRoleFullAccess-TrascribeRoleForPostCall"
client = boto3.client(service_name='transcribe')

response = client.start_call_analytics_job(
    CallAnalyticsJobName=CallAnalyticsJobName,
    Media={
        'MediaFileUri': MediaFileUri
    },
    Settings={
        'LanguageOptions': [LanguageOptions]
    },
    ChannelDefinitions = [
        {
            "ChannelId": 1,
            "ParticipantRole": "AGENT"
        },
        {
            "ChannelId": 0,
            "ParticipantRole": "CUSTOMER"
        }
    ],
    DataAccessRoleArn=DataAccessRoleArn
)
```

IAM Role인 "AmazonTranscribeServiceRoleFullAccess-TrascribeRoleForPostCall"은 아래의 퍼미션을 가집니다.

```java
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:GetObject",
                "s3:ListBucket",
                "s3:PutObject"
            ],
            "Resource": [
                "*"
            ],
            "Effect": "Allow"
        },
        {
            "Action": [
                "kms:GenerateDataKey*",
                "kms:Decrypt"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringLike": {
                    "kms:ViaService": [
                        "s3.*.amazonaws.com"
                    ]
                }
            },
            "Effect": "Allow"
        }
    ]
}
```

이것의 Trust Relationship은 아래와 같습니다.

```java
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "transcribe.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

### Post-call 결과 확인 

이때의 응답의 형태는 아래와 같습니다. response['Transcript'][0] 아래에 Content로 대화의 내용을 ParticipantRole로 대화의 상대를 확인할 수 있습니다.

![image](https://github.com/kyopark2014/LLM-post-call/assets/52392004/5cf05d92-ccfc-4fec-99c0-febd1b9a5e85)

이것을 구현하면 아래와 같습니다.

```python
response = client.get_call_analytics_job(
    CallAnalyticsJobName=CallAnalyticsJobName
)
TranscriptFileUri = response['CallAnalyticsJob']['Transcript']['TranscriptFileUri']

import requests

result = requests.get(TranscriptFileUri).json()
transcript = result['Transcript']

content = ""
for t in transcript:
    content = content+f"{t['ParticipantRole']}: {t['Content']}\n"

return content
```

이때의 결과는 아래와 같습니다.

```text
AGENT: Speaking. How can I help you?
CUSTOMER: Yeah. Hi, Terry. Um, my name is Suzanne. I literally just left your shop. Um, I just went in and got my service. Um, it just, it just like filter changes, oil change and all that kind of stuff. Um, but then I left in my oil light is still on and I don't know why.
AGENT: Mhm. Got it. You just got it service here. But when you drove off the light was still on. Is that what happened?
CUSTOMER: Yeah. Yeah. Yeah. Like I literally like I had a appointment and I just got home.
AGENT: Got it. You know, believe it or not, this, this happens not infrequently as sometimes the voice do the service and they just forget some of these cars have a little switch on the inside, basically that just kind of resets the light.
CUSTOMER: Hmm. Okay.
AGENT: And sometimes if, if they're in a hurry and there's a line we've been busy all day, they just forget to switch that little light. So, what they've done is they've done the service properly and your car is totally safe to drive, but they probably just forgot to put that little switch which turns the light off.
CUSTOMER: Oh.
AGENT: So let's, let's get that fixed for you.
CUSTOMER: Oh, okay. Okay. That makes sense.
AGENT: Are you close enough that you prefer to just turn back around and we can switch it off for you or have you already gotten home? What's easiest for you?
CUSTOMER: Um, I, I'm already home but I drive right by you all on my way to and from work.
AGENT: Okay.
CUSTOMER: So it's not really a big deal stopping.
AGENT: Okay. Well, you know what, let's see if we can make this as easy as possible for you. Um, what's the make and model of the car?
CUSTOMER: Oh. yeah, it's a Hyundai. Uh, oh, I'm sorry, I really got a car stuff.
AGENT: Uh huh. No problem.
CUSTOMER: Um, and yeah, and it's, uh, it's a 2019 and I, yeah, I bought it from you all um, last year.
AGENT: No problem. 2019. Perfect. Yes. Okay, Suzanna. Give me just one second here and I'm gonna look into the manual on that card real quick.
CUSTOMER: Sure.
AGENT: One second. Okay, Susan, are you still there?
CUSTOMER: Mhm.
AGENT: Okay. So if you would like, I am happy to walk you through, resetting that little button yourself.
CUSTOMER: Yes.
AGENT: It's actually really easy on the 2019 Elantra.
CUSTOMER: Oh, okay.
AGENT: It's actually really simple. So if you want or if you, if you're able to just walk out to the car, I can just walk you through what to do to reset that.
CUSTOMER: Yeah.
AGENT: Does that work for you?
CUSTOMER: Yeah. Yeah. Give me one, give me one second.
AGENT: Okay.
CUSTOMER: Okay. All right. Where, where am I starting the inside of the outside of the car?
AGENT: You're gonna sit just like you're gonna drive the car, okay?
CUSTOMER: Okay.
AGENT: And then you're gonna go ahead and turn it on, you'll see the light come on.
CUSTOMER: Okay. Okay.
AGENT: Okay? So now you're gonna see that light that came on below that and a little bit to the right there are kind of two tiny buttons. They'll remind you of the buttons when you have to reset your radio clock, sort of like a minute and a second on those two tiny ones.
CUSTOMER: Yeah.
AGENT: So, what you're gonna wanna do is you're gonna wanna hold them both down for about 10 seconds.
CUSTOMER: Uh huh. Yep. Mhm.
AGENT: So it'll, it'll feel a little bit long, but go ahead and stay on the Yes.
CUSTOMER: So like at the same time.
AGENT: So stay on the phone with me.
CUSTOMER: Bye.
AGENT: Go ahead and hold both of those little buttons and press them in and just count to 10 in your head and then we'll see what happens with that like you might have to turn the car back off
CUSTOMER: Mhm. Mm. Okay. Now I'm gonna try to start off and then audience.
AGENT: when you like all the.
CUSTOMER: Yeah.
AGENT: Oh it did.
CUSTOMER: No it works.
AGENT: Okay, great.
CUSTOMER: Yeah. Uh huh. Yeah it works your uh your coworker. Okay. That's great. Thank you so much. You saved me a trip.
AGENT: Oh, you're welcome. It's no problem and and next time you can just like check it right before you leave and if they've forgotten to switch it, we'll just do it for you really quick.
CUSTOMER: Sure.
AGENT: But um but yeah it's a quick fix on that guy, so.
CUSTOMER: Okay. Okay.
AGENT: Alright is there you're welcome.
CUSTOMER: Thank you so much. I appreciate your time.
AGENT: Anything else I can do for you today?
CUSTOMER: Uh no ma'am that's it.
AGENT: Okay. Well you have a good one.
CUSTOMER: Alright you too. Bye.
AGENT: Alright bye bye.
```

## Reference
[boto3 - start_transcription_job](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/transcribe/client/start_transcription_job.html#)

[boto3 - get_call_analytics_job](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/transcribe/client/get_call_analytics_job.html)

[boto3 - get_transcription_job](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/transcribe/client/get_transcription_job.html)

[boto3 - start_job](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/amplify/client/start_job.html#)

[boto3 - start_call_analytics_job](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/transcribe/client/start_call_analytics_job.html)
