# Smart Office Single sign-on (Authorization Code)

### ขั้นตอนการเรียกใช้งาน
1. กรณี Mobile app เปิดหน้า WebView , กรณี Web ให้ Redirect URL ไปที่ <URL-SSO>/oauth2/authorize โดยส่ง GET parameter ดังนี้
    
    Staging: https://logindev.mea.or.th/oauth2/authorize

    Production: https://login.mea.or.th/oauth2/authorize
    
    *  `response_type` ระบุค่า `code`
    *  `client_id` ระบุค่าตามที่ได้รับจาก กฟน. เช่น `TUVBIENvbm5leHQ=`
    *  `redirect_uri` คือ URL ของ Web App ที่ลงทะเบียนไว้กับ กฟน. `http://localhost/test`
    
    **Example:** `https://logindev.mea.or.th/oauth2/authorize?response_type=code&client_id=TUVBIENvbm5leHQ=&redirect_uri=http://localhost/test`
    
    [Error response](../README.md#authorization-error)
    

2. ผู้ใช้งานกรอกข้อมูล Login (Username/Password)

3. กรณี Mobile app รอฟังการ Redirect ผ่าน URL schema , กรณี Web SSO จะส่ง GET parameter ไปกับ URL ดังนี้
    *  `code` คือ Authorization code ที่ได้จากระบบ SSO ซึ่งจะนำไปใช้ในขั้นตอนขอ Access Token ในขั้นตอนถัดไป

    [Error response](../README.md#authorization-error)

4. Exchange `code` to `access_token` โดยส่ง HTTP POST request มาที่ /oauth2/token โดยมีรายละเอียดดังนี้
    
    Staging: https://logindev.mea.or.th/oauth2/token

    Production: https://login.mea.or.th/oauth2/token
    
    *  **Content-Type:** x-www-form-urlencoded
    *  **Parameter:**
        *  `grant_type` ระบุค่า `authorization_code`
        *  `code` คือ Authorization code ที่ได้จากขั้นตอนก่อนหน้า
        *  `client_id` ระบุค่าตามที่ได้รับจาก กฟน. เช่น `TUVBIENvbm5leHQ=`
        *  `client_secret` ระบุค่าตามที่ได้รับจาก กฟน. เช่น `jWrCUnXbAvWB9K5yVK2Y4HTSyVVnWs5hMoSq`
        *  `redirect_uri` คือ URL ของ Web App ที่ลงทะเบียนไว้กับ กฟน. `http://localhost/test`
        
    > ในขั้นตอนนี้ จะต้อง request ด้วย application ฝั่ง backend เนื่องจากจะต้องเก็บ client_secret ไว้เป็นความลับ

```js
// Example      
var axios = require('axios');
var qs = require('qs');
var data = qs.stringify({
'grant_type': 'authorization_code',
'code': '<Authorization code ที่ได้จากขั้นตอนก่อนหน้า>',
'client_id': 'TUVBIENvbm5leHQ=',
'client_secret': 'jWrCUnXbAvWB9K5yVK2Y4HTSyVVnWs5hMoSq',
'redirect_uri': 'http://localhost/test' 
});
var config = {
  method: 'post',
  url: 'https://logindev.mea.or.th/oauth2/token',
  headers: { 
    'Content-Type': 'application/x-www-form-urlencoded'
  },
  data : data
};

axios(config)
.then(function (response) {
  console.log(JSON.stringify(response.data));
})
.catch(function (error) {
  console.log(error);
});
```

    Response Parameter
    
| Name | Type | Description |
|------|------|-------------|
| access_token | string | Your access token เพื่อดึงข้อมูลพนักงาน | 
| refresh_token | string | refresh token เพื่อสร้าง access token ใหม่ | 
| expires_in | number | ระยะเวลาใช้งาน access token เช่น `1800` | 
| token_type | string | ประเภท token เช่น `Bearer` | 

```json
/*Example Response*/
{
    "access_token": "01234593ABGcXqKOQBk0hawdnBCuyqRG12345",
    "refresh_token": "012345dPWABsFSrk2NcjetpUSgqDJD5n3mvQyGdfNnUYBiymQEIC3nF5RE267890",
    "expires_in": 1800,
    "token_type": "Bearer"
}
```

[Error response](../README.md#authorization-error)

5. Web app ขอ User profile โดยการส่ง HTTP GET request มาที่ /profile โดยใส่ Access token ใน Authorization request header แบบ Bearer
   
    Staging: https://logindev.mea.or.th/profile

    Production: https://login.mea.or.th/profile

    Request header:

   - Authorization: Bearer `<Your access token ที่ได้รับจาก ข้อ 4>` 

```js
// Example
await axios.get(`https://logindev.mea.or.th/profile`, {
  headers: {
    Authorization: `Bearer <Your access token>`
  }
});
```

    Response Parameter
    
| Name | Type | Description |
|------|------|-------------|
| uuid | string | UUID ของพนักงาน | 
| empId | string | รหัสพนักงาน เช่น 2071710 | 
| prefix | string | คำนำหน้า | 
| firstName | string | ชื่อ | 
| lastName | string | นามสกุล | 
| nickName | string | ชื่อเล่น | 
| birthDate | string | วันเกิด รูปแบบ YYYY-MM-DD | 
| startDate | string | วันเริ่มงาน YYYY-MM-DD | 
| cLevel | string | ระดับ C | 
| orgId | number | รหัสหน่วยงานที่สังกัด เช่น 1667 | 
| orgName | string | ชื่อเต็มหน่วยงานที่สังกัด | 
| orgShortName | string | ชื่อย่อหน่วยงานที่สังกัด | 
| jobId | number | รหัสตำแหน่ง | 
| jobName | string | ชื่อเต็มตำแหน่ง | 
| jobShortName | string | ชื่อย่อตำแหน่ง | 
| posId | number | รหัสอัตรา | 
| posName | string | ชื่อเต็มอัตรา | 
| posShortName | string | ชื่อย่ออัตรา | 
| pathId | number | Org ID ของสายงาน | 
| pathName | string | ชื่อเต็มสายงาน | 
| pathShortName | string | ชื่อย่อสายงาน | 
| assistId | number | Org ID ของ ชผวก. | 
| assistName | string | ชื่อเต็ม ชผวก. | 
| assistShortName | string | ชื่อย่อ ชผวก. | 
| depId | number | Org ID ของฝ่าย | 
| depName | string | ชื่อเต็มฝ่าย | 
| depShortName | string | ชื่อย่อฝ่าย | 
| divId | number | Org ID ของกอง | 
| divName | string | ชื่อเต็มกอง | 
| divShortName | string | ชื่อย่อกอง | 
| secId | number | Org ID ของแผนก | 
| secName | string | ชื่อเต็มแผนก | 
| secShortName | string | ชื่อย่อแผนก | 
| partId | number | Org ID ของส่วนงาน | 
| partName | string | ชื่อเต็มส่วนงาน | 
| partShortName | string | ชื่อย่อส่วนงาน | 
| email | string | E-mail address ทางการ (*@mea.or.th) | 
| emailOptional | string | E-mail address ที่ผู้ใช้งานกำหนดเอง | 
| tel | string | เบอร์โทรศัพท์สายตรง หรือ เบอร์โทรกลาง | 
| telExtension | string | เบอร์ต่อ | 
| telInternalPrefix | string | เบอร์สังกัดเขต | 
| telInternalSuffix | string | เบอร์สายใน | 
| mobile | string | เบอร์โทรศัพท์มือถือ | 
| lineId | string | Line ID | 
| isMobilePublic | boolean | เปิดเผยเบอร์โทรศัพท์มือถือ | 
| isLineIdPublic | boolean | เปิดเผย LineID | 
| isBirthDatePublic | boolean | เปิดเผยวันเกิด | 
| isStartDatePublic | boolean | เปิดเผยวันเริ่มงาน | 
| empPicture | string | url ภาพพนักงาน | 
| profilePicture | string | url ภาพโปรไฟล์ | 
| isNormalPeriod | boolean | สถานะไม่เป็นพนักงานกะ | 
| userTypeId | number | ประเภทผู้ใช้งาน (1 = พนักงานปัจจุบัน, 2 = พนักงานเกษียณ, 3 = User ระดับฝ่าย, 4 = User ระดับแผนก) | 

```json
/*Example Response*/
{
    "meta": {
        "copyright": "Copyright 2020 The Metropolitan Electricity Authority. All rights reserved.",
        "authors": [
            "MAPBOSS Co., Ltd."
        ]
    },
    "data": {
        "empId": "2321324",
        "uuid": "A97E60A5-0587-430A-803E-FD468080D197",
        "prefix": "นาย",
        "firstName": "ภานิสิทธิ์",
        "lastName": "เฟื่องฟุ้ง",
        "nickName": "maxna",
        "birthDate": "1989-01-01",
        "startDate": "2015-01-01",
        "cLevel": "6",
        "orgId": 3944,
        "orgName": "งานพัฒนาระบบงานสนับสนุนองค์กร (ระดับ 10)",
        "orgShortName": "*",
        "jobId": 10000284,
        "jobName": "นักประมวลผลข้อมูล 6",
        "jobShortName": "นปข.6",
        "posId": 20033009,
        "posName": null,
        "posShortName": null,
        "pathId": 1678,
        "pathName": "สายงานรองผู้ว่าการเทคโนโลยีสารสนเทศฯ",
        "pathShortName": "รผส.",
        "assistId": 1679,
        "assistName": "ผู้ช่วยผู้ว่าการ (เทคโนโลยีสารสนเทศและระ",
        "assistShortName": "ชวก.",
        "depId": 1667,
        "depName": "ฝ่ายพัฒนาระบบงานประยุกต์",
        "depShortName": "ฝพป.",
        "divId": 3944,
        "divName": "งานพัฒนาระบบงานสนับสนุนองค์กร (ระดับ 10)",
        "divShortName": "*",
        "secId": null,
        "secName": null,
        "secShortName": null,
        "partId": null,
        "partName": null,
        "partShortName": null,
        "email": "panisit.fu@mea.or.th",
        "emailOptional": "panisit.fu@mea.or.th",
        "tel": "042567898",
        "telExtension": "1234",
        "telInternalPrefix": "712",
        "telInternalSuffix": "8086",
        "mobile": "0898765432",
        "lineId": "maxzalineid",
        "isMobilePublic": true,
        "isLineIdPublic": true,
        "isBirthDatePublic": true,
        "isStartDatePublic": false,
        "empPicture": "https://apidev.mea.or.th/api-content/employee/emp/A97E60A5-0587-430A-803E-FD468080D197/picture",
        "profilePicture": "https://logindev.mea.or.th/profile/A97E60A5-0587-430A-803E-FD468080D197/picture",
        "isNormalPeriod": true,
        "userTypeId": 1
    }
}
```

[ข้อแนะนำในการใช้งาน](../README.md#ข้อแนะนำในการใช้งาน)

[Error response](../README.md#resources-error)
