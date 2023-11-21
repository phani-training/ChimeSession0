# ChimeSession0
```
#include <windows.h>
#include <iostream>

void SaveBitmap(HBITMAP hBitmap, LPCSTR filename) {
    BITMAP bmp;
    PBITMAPINFO pbmi;
    WORD cClrBits;
    HDC hDC;
    HANDLE hf; // file handle
    BITMAPFILEHEADER hdr; // bitmap file header
    PBITMAPINFOHEADER pbih; // bitmap info header
    LPBYTE lpBits; // memory pointer
    DWORD dwTotal; // total count of bytes
    DWORD cb; // incremental count of bytes
    BYTE *hp; // byte pointer
    DWORD dwTmp;

    hDC = CreateCompatibleDC(GetWindowDC(GetDesktopWindow()));
    SelectObject(hDC, hBitmap);

    GetObject(hBitmap, sizeof(BITMAP), (LPSTR)&bmp);

    cClrBits = (WORD)(bmp.bmPlanes * bmp.bmBitsPixel);
    if (cClrBits == 1) {
        cClrBits = 1;
    } else if (cClrBits <= 4) {
        cClrBits = 4;
    } else if (cClrBits <= 8) {
        cClrBits = 8;
    } else if (cClrBits <= 16) {
        cClrBits = 16;
    } else if (cClrBits <= 24) {
        cClrBits = 24;
    } else {
        cClrBits = 32;
    }

    if (cClrBits != 24) {
        pbmi = (PBITMAPINFO)LocalAlloc(LPTR, sizeof(BITMAPINFOHEADER) + sizeof(RGBQUAD) * (1 << cClrBits));
    } else {
        pbmi = (PBITMAPINFO)LocalAlloc(LPTR, sizeof(BITMAPINFOHEADER));
    }

    pbih = (PBITMAPINFOHEADER)pbmi;
    pbih->biSize = sizeof(BITMAPINFOHEADER);
    pbih->biWidth = bmp.bmWidth;
    pbih->biHeight = bmp.bmHeight;
    pbih->biPlanes = bmp.bmPlanes;
    pbih->biBitCount = bmp.bmBitsPixel;
    if (cClrBits < 24) {
        pbih->biClrUsed = (1 << cClrBits);
    }

    pbih->biCompression = BI_RGB;
    pbih->biSizeImage = ((pbih->biWidth * cClrBits + 31) & ~31) / 8 * pbih->biHeight;
    pbih->biClrImportant = 0;

    lpBits = (LPBYTE)GlobalAlloc(GMEM_FIXED, pbih->biSizeImage);

    if (!lpBits) {
        std::cerr << "Could not allocate memory." << std::endl;
        return;
    }

    if (!GetDIBits(hDC, hBitmap, 0, (WORD)pbih->biHeight, lpBits, pbmi, DIB_RGB_COLORS)) {
        std::cerr << "GetDIBits failed." << std::endl;
        return;
    }

    hf = CreateFile(filename, GENERIC_READ | GENERIC_WRITE, (DWORD)0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, (HANDLE)NULL);
    if (hf == INVALID_HANDLE_VALUE) {
        std::cerr << "Could not create file for writing." << std::endl;
        return;
    }

    hdr.bfType = 0x4d42; // "BM"
    hdr.bfSize = (DWORD)(sizeof(BITMAPFILEHEADER) + pbih->biSize + pbih->biClrUsed * sizeof(RGBQUAD) + pbih->biSizeImage);
    hdr.bfReserved1 = 0;
    hdr.bfReserved2 = 0;
    hdr.bfOffBits = (DWORD) sizeof(BITMAPFILEHEADER) + pbih->biSize + pbih->biClrUsed * sizeof(RGBQUAD);

    if (!WriteFile(hf, (LPVOID)&hdr, sizeof(BITMAPFILEHEADER), (LPDWORD)&dwTmp, NULL)) {
        std::cerr << "Could not write file header." << std::endl;
        return;
    }

    if (!WriteFile(hf, (LPVOID)pbih, sizeof(BITMAPINFOHEADER) + pbih->biClrUsed * sizeof(RGBQUAD), (LPDWORD)&dwTmp, (NULL))) {
        std::cerr << "Could not write info header." << std::endl;
        return;
    }

    if (!WriteFile(hf, (LPSTR)lpBits, (int)pbih->biSizeImage, (LPDWORD)&dwTmp, NULL)) {
        std::cerr << "Could not write image data." << std::endl;
        return;
    }

    GlobalFree((HGLOBAL)lpBits);
    CloseHandle(hf);

    std::cout << "Screenshot saved to " << filename << std::endl;
}

int main() {
    // Capture the entire screen
    int x1 = 0, y1 = 0, x2 = GetSystemMetrics(SM_CXSCREEN), y2 = GetSystemMetrics(SM_CYSCREEN);

    HDC hScreenDC = GetDC(NULL);
    HDC hMemoryDC = CreateCompatibleDC(hScreenDC);

    int width = x2 - x1;
    int height = y2 - y1;

    HBITMAP hBitmap = CreateCompatibleBitmap(hScreenDC, width, height);

    HBITMAP hOldBitmap = (HBITMAP)SelectObject(hMemoryDC, hBitmap);

    BitBlt(hMemoryDC, 0, 0, width, height, hScreenDC, x1, y1, SRCCOPY);
    hBitmap = (HBITMAP)SelectObject(hMemoryDC, hOldBitmap);

     std::string fileName = "SampleFile" + index;
 std::string ext = ".bmp";
 fileName = fileName + ext;
 SaveBitmap(hBitmap, fileName.c_str());

    DeleteDC(hMemoryDC);
    DeleteDC(hScreenDC);
    ReleaseDC(NULL, hScreenDC);
    DeleteObject(hBitmap);

    return 0;
}

```
