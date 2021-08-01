# MEA Smart Office Single sign-on

## วิธีการ Login

- [Authorization Code (Staging)](/grant/README.md)

<br/>

## วิธีการ Logout

1. Client ส่ง HTTP POST request มาตามข้อมูลด้านล่าง

   Staging: https://logindev.mea.or.th/logout
   
   Production: https://login.mea.or.th/logout

   Request header:

   - Authorization: Bearer `<Your access token>`

```js
// Example
await axios.post(`https://logindev.mea.or.th/logout`, null, {
  headers: {
    Authorization: `Bearer <Your access token>`
  }
});
```

<br/>

## วิธีการ Forgot password

1. User กดปุ่ม "ลืมรหัสผ่าน" ในหน้า Login

2. ระบบ SSO จะนำ user ไปตาม flow การ reset password สิ้นสุดที่หน้าแสดงผลลัพธ์การส่ง link reset password เข้า email ของ user **โดยหน้าผลลัพธ์จะมีปุ่ม "เข้าสู่ระบบ"** เพื่อให้ user เข้าสู่ระบบอีกครั้ง หลัง reset password เรียบร้อยแล้ว

3. User เปิด email client และเปลี่ยนรหัสผ่าน
4. User กลับมาที่ application/website และ login ด้วย username/password ใหม่

<br/>

## วิธีการ Change password

1. Client ส่ง HTTP PUT request มาตามข้อมูลด้านล่าง

   Staging: https://logindev.mea.or.th/profile/password
   
   Production: https://login.mea.or.th/profile/password

   Request header:

   - Authorization: Bearer `<Your access token>`

   Request body:

   - `oldPassword <string>` รหัสผ่านเก่า
   - `newPassword <string>` รหัสผ่านใหม่
   - `confirmPassword <string>` ยืนยันรหัสผ่านใหม่

2. Client รอฟัง response จาก API ดังนี้

| HTTP status codes | Error codes               | Description                                |
| ----------------- | ------------------------- | ------------------------------------------ |
| 200               | -                         | เปลี่ยนรหัสผ่านสำเร็จ                      |
| 400               | MISSING_OLD_PASSWORD      | ไม่ส่งรหัสผ่านเก่า                         |
| 400               | MISSING_NEW_PASSWORD      | ไม่ส่งรหัสผ่านใหม่                         |
| 400               | MISSING_CONFIRM_PASSWORD  | ไม่ส่งยืนยันรหัสผ่านใหม่                   |
| 400               | MISMATCH_CONFIRM_PASSWORD | รหัสผ่านใหม่และยืนยันรหัสผ่านใหม่ไม่ตรงกัน |
| 400               | INVALID_PASSWORD          | รหัสผ่านเก่าไม่ถูกต้อง                     |
| 500               | -                         | กรณี error อื่น ๆ                          |

<br/>

## ข้อแนะนำในการใช้งาน

- ควรเรียก API get profile (/oauth2/profile) ทุกครั้งที่มีการเปิด application หรือ refresh หน้าเว็บไซต์ เพื่อเป็นการ update ข้อมูล profile ล่าสุด
- กรณีเรียก API ที่เกี่ยวข้องกับ resource ของผู้ใช้งาน และได้ผลลัพธ์ **_HTTP Status Code 401 Unauthorized ทุก application หรือเว็บไซต์ต้องทำการ logout ผู้ใช้งานออกจากระบบ_**

<br/>

## Error handler

Error ที่เกิดจากการเรียกใช้งาน SSO จะมี 2 ลักษณะ คือ

[1. Error ที่ต้อง handle](#1.-Error-ที่ต้อง-handle)

[2. Error สำหรับแสดงผล](#2.-Error-สำหรับแสดงผล)

### 1. Error ที่ต้อง handle

ในกรณีนี้ client ที่เรียกใช้ SSO รอฟังการ throw error โดย redirect ผ่าน URL ที่มี parameter `error` ในรูปแบบ `<redirect_uri>/#error=<error_code>` เช่น `http://localhost:3000/#error=access_denied`

โดย error_code และวิธีการ handle เป็นดังนี้

##### `access_denied`

##### สาเหตุที่เกิด:

- User กดไม่อนุญาตในหน้า request permission

##### วิธีการ Handle error

วิธีการ Handle error แต่ละ platform ไม่บังคับว่าเป็นอย่างไร ขึ้นอยู่กับ business flow ของ client application แต่รายการที่แนะเป็นดังนี้

- Mobile

  - แสดงข้อความ error ตาม business flow ที่ต้องการ
  - ปิดหน้าจอ Login

- Web
  - แสดงข้อความ error ตาม business flow ที่ต้องการ
  - Redirect กลับไปหน้าหลัก

### 2. Error สำหรับแสดงผล

ในกรณีนี้เป็น error อื่น ๆ ที่ไม่ได้เกิดจากการกระทำของผู้ใช้งาน จึงไม่จำเป็นต้องมีการ handle error ต่าง ๆ การแสดงผล error จะทำโดยระบบ SSO โดย error_code เป็นดังนี้

#### Authorization error

##### `invalid_request`

##### `unauthorized_client`

##### `access_denied`

##### `unsupported_response_type`

##### `invalid_scope`

##### `temporarily_unavailable`

#### Token error

##### `invalid_request`

##### `invalid_client`

##### `invalid_grant`

##### `unauthorized_client`

##### `unsupported_grant_type`

##### `invalid_scope`


### Resources error

เกิดขึ้นกรณีที่เรียกข้อมูล resource เช่น ข้อมูล profile, profile picture ไม่สำเร็จ โดยการ throw error จะอยู่ในรูปแบบ JSON

```json
{
    "meta": <object>,
    "errors": [
        {
            "code": <string>,
            "title": <string>
        }
    ]
}
```

#### Error code

`UNAUTHORIZED`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Access Token ไม่ถูกต้อง หรือไม่พบ Access Token ใน HTTP request headers

`ACCESS_TOKEN_EXPIRED`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Access Token หมดอายุ

<br/>

## วิธีการ Refresh Access Token

ส่ง HTTP POST request มาที่ /oauth2/token โดยมีรายละเอียดดังนี้

Staging: https://logindev.mea.or.th/oauth2/token

Production: https://login.mea.or.th/oauth2/token

- **Content-Type:** x-www-form-urlencoded
- **Parameter:**
  - `grant_type` ระบุค่า `refresh_token`
  - `refresh_token` คือ Refresh Token
  - `client_id` ระบุค่าตามที่ได้รับจาก กฟน. เช่น `TUVBIENvbm5leHQ=`
  - `client_secret` ระบุค่าตามที่ได้รับจาก กฟน. เช่น `jWrCUnXbAvWB9K5yVK2Y4HTSyVVnWs5hMoSq`

    > ในขั้นตอนนี้ จะต้อง request ด้วย application ฝั่ง backend เนื่องจากจะต้องเก็บ client_secret ไว้เป็นความลับ

```js
// Example      
var axios = require('axios');
var qs = require('qs');
var data = qs.stringify({
'grant_type': 'refresh_token',
'refresh_token': '<refresh_token ที่ได้รับจาก oauth2/token>',
'client_id': 'TUVBIENvbm5leHQ=',
'client_secret': 'jWrCUnXbAvWB9K5yVK2Y4HTSyVVnWs5hMoSq'
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

[Error response](#token-error)
