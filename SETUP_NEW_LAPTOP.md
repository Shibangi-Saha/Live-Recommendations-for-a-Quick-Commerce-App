# Run the whole demo on another (Windows) laptop

Needs: Windows 10 or 11, 8 GB RAM or more, 10 GB free disk, internet, and about 45 minutes.
Do this the day before, not on the day.

## 1. Install Python
Download Python 3.12 from python.org. In the installer TICK "Add python.exe to PATH". Open a new PowerShell and check:
    python --version

## 2. Install Ubuntu (this is where Kafka runs)
Open PowerShell as ADMINISTRATOR:
    wsl --install -d Ubuntu
Restart when asked. Open "Ubuntu" from the Start menu, create a username and password (remember them), then close it.
(If it says virtualization is disabled, it must be switched on in the laptop's BIOS.)

## 3. Copy the project
Unzip `qc_streaming_FINAL.zip` so you get `C:\Users\<you>\Desktop\qc`.
Copy the `.env` file from your own laptop into that `qc` folder (USB stick; do not email it, it has your Atlas password).

## 4. Allow this laptop in Atlas
In Atlas: Network Access > Add IP Address > "Add current IP address". Do it again if you change network.

## 5. Python packages (PowerShell, inside the qc folder)
    cd $HOME\Desktop\qc
    python -m venv venv
    Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
    venv\Scripts\Activate.ps1
    python -m pip install -r requirements.txt

## 6. Kafka (Ubuntu window, one time)
Open Ubuntu from the Start menu and run (replace WINUSER with your Windows username):
    cd /mnt/c/Users/WINUSER/Desktop/qc
    bash setup_kafka_wsl.sh
It asks for your Ubuntu password once. Wait for "DONE", then close Ubuntu.

## 7. Start the demo
In the qc folder double-click `start_demo.bat` (or run `.\start_demo.bat` in PowerShell).
It opens 6 black windows. Wait about 6 minutes, then open Chrome:
    Dashboard:    http://127.0.0.1:8050
    Shop (live):  http://127.0.0.1:8000/?group=treatment
    Shop (stale): http://127.0.0.1:8000/?group=control
Windows may ask to allow Python/Java through the firewall: click Allow.

## 8. Test checklist
- [ ] Dashboard shows numbers (about 20 events per second).
- [ ] Shop says "connected". Add Milk: three recommendations with reasons.
- [ ] Stale page shows its list and "snapshot ... ago".
- [ ] Place an order: cart clears. Dashboard funnel moves.
- [ ] Burst: in a new PowerShell in qc, run `venv\Scripts\python.exe m2_trend_burst.py --product-id 272170`, wait a few seconds, open a fresh live page: Ice Cream shows as Trending.
- [ ] Dashboard header shows no red "some queries failed".

## 9. If something fails
| You see | Fix |
| --- | --- |
| "Could not connect to Kafka" | The Kafka window is not running or not ready. Wait 30 seconds and run the script again, or check the Kafka window for errors |
| "Could not reach MongoDB Atlas" | Add this laptop's IP in Atlas Network Access; check `.env` |
| `wsl` is not recognized | WSL is not installed: redo step 2 |
| venv or pip errors | Redo step 5 in a NEW PowerShell, make sure `python --version` works |
| The dashboard is empty | The window "A consumer" is not running |
| No recommendations | The window "B processor" is not running |

## 10. After the demo
Close the black windows and run `wsl --shutdown`. Remove this laptop's IP from Atlas Network Access if you want.

## Plan B
Keep the recorded video and screenshots on a USB stick.
