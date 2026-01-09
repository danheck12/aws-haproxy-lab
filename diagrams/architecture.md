## Architecture

Client (Internet)
   |
   v
[ lb-01 (HAProxy, public) ]
   |
   +--> [ web-01 (Nginx) ]
   |
   +--> [ web-02 (Nginx) ]
