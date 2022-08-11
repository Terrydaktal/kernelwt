// ConsoleApplication3.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include <iostream> //required for std::cout
#include <Windows.h> //required for for function and structure type definitons and preprocessor definitions
#include <setupapi.h> //required for SetupOpenInfFile and SetupInstallFileW type defintions

#define BENCHMARK_DRV_IOCTL CTL_CODE(FILE_DEVICE_UNKNOWN, 0x902, METHOD_NEITHER, FILE_ANY_ACCESS)

int InstallAndStartDriver(void);
LPCSTR AppName;
PVOID ExceptionAddress;

int InstallAndStartDriver() {

	UINT ErrorLine;
	PCWSTR InfFileName = L"C:\\Users\\lewis\\source\\repos\\ConsoleApplication3\\x64\\Release\\kernelwt.inf";
	PCWSTR DriverName = L"kernelwt";
	PCWSTR SourceFile = L"kernelwt.sys";
	PCWSTR SourcePathRoot = L"C:\\Users\\lewis\\source\\repos\\ConsoleApplication3\\x64\\Release\\";
	PCWSTR DriverInstallPath = L"C:\\Users\\lewis\\source\\repos\\ConsoleApplication3\\x64\\Release\\kernelwt.sys";
	LPCSTR SubKey = "System\\CurrentControlSet\\Services\\kernelwt";
	HKEY hKey;
	DWORD ErrorControl = 1;
	DWORD StartType = 3;
	PBOOL FileWasInUse = NULL;
	LSTATUS status;

	if (status = RegCreateKeyExA(HKEY_LOCAL_MACHINE,
		SubKey,
		NULL, NULL,
		REG_OPTION_NON_VOLATILE,
		KEY_WRITE,
		NULL,
		&hKey,
		NULL))
	{
		return 1;
	}
	else {
		if (RegSetValueEx(hKey, L"DriverName", NULL, REG_SZ, (LPBYTE)DriverName, sizeof(wchar_t)*(wcslen(DriverName) + 1))
			|| RegSetValueEx(hKey, L"DisplayName", NULL, REG_SZ, (LPBYTE)DriverName, sizeof(wchar_t)*(wcslen(DriverName) + 1))
			|| RegSetValueEx(hKey, L"ErrorControl", NULL, REG_DWORD, (LPBYTE)&ErrorControl, sizeof(DWORD))
			)
		{
			return 2;
		}
	}

	if (SC_HANDLE manager = OpenSCManager(NULL, NULL, SC_MANAGER_ALL_ACCESS)) {

		HINF HInf = SetupOpenInfFile(InfFileName, NULL, INF_STYLE_WIN4 | INF_STYLE_OLDNT, &ErrorLine);
		if (SetupInstallFileW(HInf, NULL, SourceFile, SourcePathRoot, DriverInstallPath,
			SP_COPY_NEWER_OR_SAME, NULL, FileWasInUse)) {

			SC_HANDLE service;
			service = CreateService(manager,
				DriverName,
				DriverName,
				SERVICE_ALL_ACCESS,
				SERVICE_KERNEL_DRIVER,
				SERVICE_DEMAND_START,
				SERVICE_ERROR_NORMAL,
				DriverInstallPath,
				NULL, NULL, NULL, NULL, NULL);

			if (GetLastError() == ERROR_SERVICE_EXISTS) { //1073
				service = OpenService(manager, DriverName, SERVICE_ALL_ACCESS);
			}

			status = StartService(service, NULL, NULL);
			DWORD error = GetLastError();
			return  error == ERROR_SERVICE_ALREADY_RUNNING | status == TRUE ? 0 : error; //1056
		}

		else {
			return 4;
		}
	}
	else {
		return 5;
	}
	return 0;
}



int main(int argc, CHAR* volatile * argv)
{
	HANDLE hDevice;
	PCWSTR SymLink = L"\\\\.\\kernelwt";
	DWORD bytesReturned;
	HMODULE hApp;
	PIMAGE_NT_HEADERS64 PEHeader;

	//sanity check of parameters

	//kernelwt waittime functions addresses lengths idabase modulebase process

	if (argc != 8) {   //if fewer than 3 arguments
		std::cout << "invalid usage, use /? for help";
		return false;
	}

	//if (int i = InstallAndStartDriver()) {  //if driver not installed, install; if driver not started, start
	//	std::cout << "error" << i;        //if error during install start / install check
	//	return false;
	//};

	int result = 1;

	FILE *functionsfile, *addressesfile, *lengthsfile;

	fopen_s(&functionsfile, argv[2], "r");
	fopen_s(&addressesfile, argv[3], "r");
	fopen_s(&lengthsfile, argv[4], "r");

	if (!functionsfile || !addressesfile || !lengthsfile) { return false; };
	char *endptr;
	char line[20];
	int count =0;
	while (fgets(line, 20, functionsfile)) count++;


	rewind(functionsfile);

	char** functions  = (char**) calloc(count, sizeof(char*));
	unsigned long long* addresses = (unsigned long long*)calloc(count, sizeof(unsigned long long));
	unsigned long long* lengths = (unsigned long long*)calloc(count, sizeof(unsigned long long));

	count =0;

	while (fgets(line, 20, functionsfile))
	{
		functions[count] = line;
		count++;
	}

	count = 0;

	while (fgets(line, 20, addressesfile))
	{
		addresses[count] = strtoull(line, &endptr, 16);
		count++;
	}

	while (fgets(line, 20, addressesfile))
	{
		lengths[count] = strtoull(line, &endptr, 16);
		count++;
	}


	//allocate and zero the output buffer on the heap
	size_t bufferSize = sizeof(unsigned long long)*argc;
	unsigned long long* bufferOut = (unsigned long long*) calloc(argc, sizeof(unsigned long long));

	//lock the output and input buffer into memory so that there isn't a page fault during execution
	//which will hang the system at IRQL >2
	result &= VirtualLock(bufferOut, bufferSize);
	result &= VirtualLock((LPVOID)argv, bufferSize);

}

