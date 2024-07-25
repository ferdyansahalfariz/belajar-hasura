action definition

```
type Query {
  """
  inquiryCardData
  """
  inquiryCardData(request: RequestCardData!): ResponseCardData
}
```

type config

```
input RequestCardData {
  cardNo: String!
}

type ResponseCardData {
  status: String
  responseCode: String
  responseDesc: String
  responseData: ResponseCardDataWrapper
}

type ResponseCardDataWrapper {
  cardNo: String
  branchCode: String
  typeCode: String
  cardStatus: String
  name: String
  address1: String
  address2: String
  city: String
  province: String
  postalCode: String
  countryCode: String
  phoneNo: String
  motherName: String
  employer: String
  idType: String
  idNumber: String
  gender: String
  maritalStatus: String
  birthDate: String
  accNo: String
  expDate: String
  mbNo: String
  mbStatus: String
  cardType: String
  type: String
  issueDate: String
  activeDate: String
  lastMaint: String
  userMaint: String
  blockDate: String
  userBlock: String
  statusDesc: String
}
```

sample input

```
{
  "action": {
    "name": "inquiryCardData"
  },
  "input": {
    "request": {
      "cardNo": "cardNo"
    }
  }
}
```

config respon body

```
{{ if $body.status == "200" }}
{
  "status": {{$body.status}},
  "responseCode": {{$body.responseCode}},
  "responseDesc": {{$body.responseDesc}},
  "data": {
    "cardNo": {{$body.responseData.cardNo}},
    "branchCode": {{$body.responseData.branchCode}},
    "typeCode": {{$body.responseData.typeCode}},
    "cardStatus": {{$body.responseData.cardStatus}},
    "name": {{$body.responseData.name}},
    "address1": {{$body.responseData.address1}},
    "address2": {{$body.responseData.address2}},
    "city": {{$body.responseData.city}},
    "province": {{$body.responseData.province}},
    "postalCode": {{$body.responseData.postalCode}},
    "countryCode": {{$body.responseData.countryCode}},
    "phoneNo": {{$body.responseData.phoneNo}},
    "motherName": {{$body.responseData.motherName}},
    "employer": {{$body.responseData.employer}},
    "idType": {{$body.responseData.idType}},
    "idNumber": {{$body.responseData.idNumber}},
    "gender": {{$body.responseData.gender}},
    "maritalStatus": {{$body.responseData.maritalStatus}},
    "birthDate": {{$body.responseData.birthDate}},
    "accNo": {{$body.responseData.accNo}},
    "expDate": {{$body.responseData.expDate}},
    "mbNo": {{$body.responseData.mbNo}},
    "mbStatus": {{$body.responseData.mbStatus}},
    "cardType": {{$body.responseData.cardType}},
    "type": {{$body.responseData.type}},
    "issueDate": {{$body.responseData.issueDate}},
    "activeDate": {{$body.responseData.activeDate}},
    "lastMaint": {{$body.responseData.lastMaint}},
    "userMaint": {{$body.responseData.userMaint}},
    "blockDate": {{$body.responseData.blockDate}},
    "userBlock": {{$body.responseData.userBlock}},
    "statusDesc": {{$body.responseData.statusDesc}}
  }
}
{{ else }}
{
  "message": {{$body.responseDesc}},
  "extensions": {
    "code": {{$body.status}}
  }
}
{{ end }}
```

# brimo pangkalan

```
type Query {
  """
  Hit API getIdKeagenan dari BRIMOLA
  """
  inquiryBrimolaPangkalan(
    keagenanInput: ParamIdPangkalan!
  ): ResponseAgenPangkalan
}
```

type config

```
input ParamIdPangkalan {
  pangkalanId: String!
}

type ResponseAgenPangkalan {
  rc: String
  desc: String
  data: DataPangkalan
}

type DataPangkalan {
  agenId: String
  agenName: String
  pangkalanId: String
  pangkalanName: String
  agenAlamat: String
  agenNorek: String
  corpBriva: String
  corpClientBriva: String
}

```

sample input

```
{
  "action": {
    "name": "inquiryBrimolaPangkalan"
  },
  "input": {
    "keagenanInput": {
      "pangkalanId": "pangkalanId"
    }
  }
}
```

