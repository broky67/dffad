[Unit]
Description=Xml to Bin converter service
After=network.target

[Service]
Type=notify
WorkingDirectory=/media/sf_UbuntuShare/XmlConverter/
ExecStart=/usr/bin/dotnet /media/sf_UbuntuShare/XmlConverter/Pilot.HwTool.Service.dll
Restart=always
RestartSec=10
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_ROOT=/usr/share/dotnet

# Права пользователя (замените `your_user` на актуальный)
User=your_user
Group=your_user

# Важно для доступа к shared-папкам
PrivateTmp=false
ProtectSystem=false

[Install]
WantedBy=multi-user.target
