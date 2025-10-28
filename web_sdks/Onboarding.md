# Onboarding

## Overview

The onboarding flow is designed for new users to complete their identity verification process. This includes document capture, face liveness checks, and identity validation.

## Flow Details

During onboarding, the user will be guided through the following steps:

1. Document capture (ID, passport, etc.)
2. Face liveness verification
3. Identity checks against government databases
4. Data pooling for enhanced verification

## Result Event Payload for Onboarding

The `result` event for onboarding includes the following data structure:

**Common Fields:**

- `data`: The result data from the API or pooling service
- `flow`: "ONBOARDING"
- `userId`: The unique identifier for the user
- `endpoint`: The endpoint used for the request

### Example Payload

```json
{
  "data": {
    {
      "id": 0,
      "created_at": "Sat, 25 Jul 2020 15:40:00 GMT",
      "company": "Acc_Demo",
      "fullname": "CLIENT_NAME",
      "birth": "Fri, 03 Jul 1998 00:00:00 GMT",
      "document_type": "national-id",
      "document_number": "CLIENT_DOCUMENT_NUMBER",
      "verification": {
        "alteration": true,
        "estimated_age": true,
        "face_match": true,
        "ip_validation": true,
        "liveness": true,
        "liveness_doc": null,
        "one_to_many_result": true,
        "template": true,
        "ubica": null,
        "verification_status": "completed",
        "watch_list": true
      },
      "blacklist": {
        "result": false,
        "score": 0.8766211271286011,
        "id": 556885
      },
      "comply_advantage": {
        "comply_advantage_result": null,
        "comply_advantage_url": null
      }
    }
  },
  "flow": "ONBOARDING",
  "userId": "user_12345",
  "endpoint": "{API_HOST}/api/v1/identity/{userId}"
}
```

For more detailed information on the `data` field structure, refer to the [Postman Documentation](https://documenter.getpostman.com/view/2293906/T1DtdvBk?version=latest#283c98a6-b797-48d7-ad1e-c756efca7477).
