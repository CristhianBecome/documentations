# Reverification

## Overview

The reverification flow is designed for existing users to re-verify their identity, typically for security purposes or periodic compliance checks. This process focuses on biometric verification and liveness detection.

## Flow Details

During reverification, the user will be guided through the following steps:

1. Face liveness verification with challenge-response
2. Biometric matching against stored reference data
3. Liveness confidence scoring
4. Audit trail generation

## Result Event Payload for Reverification

The `result` event for reverification includes additional fields beyond the common ones:

**Common Fields:**
- `data`: The result data from the API or pooling service
- `flow`: "REVERIFICATION"
- `userId`: The unique identifier for the user
- `endpoint`: The endpoint used for the request

**Additional Fields for REVERIFICATION:**
- `livenessConfidence`: The confidence score from Liveness (number)
- `auditImages`: Audit images from Liveness (array)
- `referenceImage`: Reference image from Liveness (object)

### Example Payload

```json
{
  "data": {
    {
      "company": "<YOUR_COMPANY_ACCOUNT_NAME>",
      "confidence": 2.4115853309631348,
      "executionId": "f850eaa8-db4e-468f-a5a3-14cffbf975c0",
      "liveness": 99.89696502685547,
      "result": false,
      "user_id": "<USER_ID>"
    }
  },
  
  "endpoint": "{API_HOST}/api/v1/matches",
  "userId": "test_1",
  "livenessConfidence": 99.91615295410156,
  "auditImages": [
    {
      "BoundingBox": {
        "Height": 283.93048095703125,
        "Left": 218.52137756347656,
        "Top": 105.67168426513672,
        "Width": 193.6444854736328
      },
      "Bytes": ""
    }
  ],
  "referenceImage": {
    "BoundingBox": {
      "Height": 287.81195068359375,
      "Left": 206.05030822753906,
      "Top": 76.31394958496094,
      "Width": 201.10231018066406
    },
    "Bytes": ""
  },
}
```

For more detailed information on the `data` field structure, refer to the [Postman Documentation](https://documenter.getpostman.com/view/2293906/T1DtdvBk?version=latest#06c03291-3e2f-4f66-bf6d-a7bf179d17df).