# zubot-TRex
#include <iostream>
#include <Windows.h>

void GetDCMem(HDC &mem) {
	HDC hdcSource = GetDC(NULL);
	mem = CreateCompatibleDC(hdcSource); //Stores "image" of desktop image specifed by a NULL in GetDC()
	HBITMAP bitmap = CreateCompatibleBitmap(hdcSource, 650, 200);
	HBITMAP oldmap = (HBITMAP)SelectObject(mem, bitmap); //Put bitmap data into data

	if (!BitBlt(mem, 0, 0, 650, 200, hdcSource, 350, 300, CAPTUREBLT | SRCCOPY)) { //Transfer color data
		std::cout << "Bitmap failed" << std::endl;
	}

	DeleteObject(oldmap);
	DeleteObject(bitmap);
	ReleaseDC(NULL, hdcSource);
}

void Jump() {
	INPUT jump = { 0 };
	jump.type = INPUT_KEYBOARD;
	jump.ki.wVk = VK_SPACE;
	SendInput(1, &jump, sizeof(INPUT));

	ZeroMemory(&jump, sizeof(jump));
	jump.ki.dwFlags = KEYEVENTF_KEYUP;
	SendInput(1, &jump, sizeof(INPUT));
}

void Crouch(int keyDelay) {
	INPUT crouch = { 0 };
	crouch.type = INPUT_KEYBOARD;
	crouch.ki.wVk = VK_DOWN;
	SendInput(1, &crouch, sizeof(INPUT));

	Sleep(keyDelay);

	crouch.ki.dwFlags = KEYEVENTF_KEYUP; //Stop crouching
	SendInput(1, &crouch, sizeof(INPUT));
}

int main() {
	std::cout << "Press BACKSPACE to start the bot" << std::endl;

	while (true) { //Start loop
		if (GetAsyncKeyState(VK_BACK)) {
			break;
		}

		Sleep(100);
	}

	std::cout << "Bot is running..." << std::endl;

	HDC mem = NULL;
	COLORREF color;

	while (true) { //Main bot loop
		GetDCMem(mem);

		if(GetAsyncKeyState(VK_ESCAPE)){
			break;
		}

		for (int i = 143; i <= 146; i++) { //Check 3 x-range pixels
			color = GetPixel(mem, 145, 54);

			if (color == 5460819) { //Bird Detected, crouch
				Crouch(350);
				break;
			}
		}

		for (int i = 142; i <= 148; i++) {
			color = GetPixel(mem, i, 84);

			if (color == 5460819) { //Object Detected, jump
				Jump();

				Sleep(250); //Max height, then crouch to reduce fall time
				Crouch(190);
				break;
			}
		}

		Sleep(10);
	}

	DeleteDC(mem);
	std::cout << "Bot has ended" << std::endl;

	std::cin.get();
	return 0;
}
