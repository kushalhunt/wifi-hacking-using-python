import subprocess
import sys
import time
from tabulate import tabulate
from colorama import Fore, Style, init

init()

def get_wifi_profiles():
    """Fetches saved Wi-Fi profile names."""
    try:
        data = subprocess.check_output(
            ['netsh', 'wlan', 'show', 'profiles'], stderr=subprocess.DEVNULL
        ).decode('utf-8').split('\n')
        return [i.split(":")[1][1:-1] for i in data if "All User Profile" in i]
    except subprocess.CalledProcessError:
        print(Fore.RED + "[Error] Failed to retrieve Wi-Fi profiles. Run as Administrator." + Style.RESET_ALL)
        sys.exit(1)

def get_wifi_password(profile):
    """Fetches the Wi-Fi password for a given profile."""
    try:
        results = subprocess.check_output(
            ['netsh', 'wlan', 'show', 'profile', profile, 'key=clear'], 
            stderr=subprocess.DEVNULL
        ).decode('utf-8').split('\n')

        
        passwords = [b.split(":")[1][1:-1] for b in results if "Key Content" in b]
        return passwords[0] if passwords else "No Password Saved"
    except subprocess.CalledProcessError:
        return "Error Fetching"

def loading_animation(message, seconds=2):
    """Displays a loading animation."""
    chars = ['|', '/', '-', '\\']
    for _ in range(seconds * 10):
        sys.stdout.write(Fore.CYAN + f"\r{message} " + chars[_ % len(chars)] + Style.RESET_ALL)
        sys.stdout.flush()
        time.sleep(0.1)
    print("\n")


loading_animation("Scanning saved Wi-Fi networks...", 3)
profiles = get_wifi_profiles()


wifi_data = []
for profile in profiles:
    wifi_data.append([profile, get_wifi_password(profile)])


print(Fore.GREEN + "\nSaved Wi-Fi Networks & Passwords:\n" + Style.RESET_ALL)
print(tabulate(wifi_data, headers=["Wi-Fi Name", "Password"], tablefmt="grid"))
