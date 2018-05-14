//Podstawowa funkcjonalność : 
//Instalator programów z bazy instalacyjnej znajdującej się w pliku ListaProgramow.cfg
//Instalacja odbywa się zgodnie z plikiem konfiguracyjnym ProgramyDoZainstalowania.cfg
//
//CO WPROWADZONO:
//-otwieranie pliku ProgramyDoZainstalowania.cfg
//-pobieranie linii z pliku (z wyjątkiem linii oznaczonych # - komentarze)
//-zamiana linii pliku na 2 stringi konfiguracyjne
//-funkcja otwierająca programy z parametrami w formie 2 stringów
//-funkcja otwierająca programy z parametrami w formie LPCTSTR nazwyprogramu i char* argumenty
//-wrzucanie stringów do kolejki
//-otwieranie pliku ProgramyDoZainstalowania.cfg
//-ustawianie parametrów itp.
//
//CZEGO NIE WPROWADZONO:
//
//
//
//
//LISTA ZMIAN :
//1.1-dodano opcję usuwania plików poprzez insName=DELETEFILES i pozostawienie pustych exec, args, path
//1.2-dodano opcję otwierania lokalizacji z plikiem (aby móc uruchomić coś jako user lokalny) poprzez wpisanie w polu exec $asuser

#include <queue>
#include <fstream>
#include <windows.h>
#include <stdio.h>
#include <string>
#include <tchar.h>
#include <iostream>

using namespace std;

//ZMIENNE GLOBALNE:
int returnFindCFG;//
int returnGetInstaller;//
string idComponent;//
string idPackage;//
string insName;//
string path;//
string exec;//
string args;//
string computerPath="c:\\Adam\\PlikiInstalacyjne\\";
const char * constPath = computerPath.c_str();
string spacer="==================";

//wyszukiwanie spacji i jej usuwanie
string SpaceEraser(string name)
{
	string cleanName="";
	for(int i=0;i<name.length();i++)
	{
		if(name[i]!=' ')
			cleanName=cleanName+name[i];
		
	}
	return cleanName;
}

//F-cja uruchamiająca program z parametrami LPCTSTR sciezka/nazwa_aplikacji, char* argumenty
void Startup(LPCTSTR lpApplicationName, char* args=NULL)
{
	
   // additional information
   STARTUPINFO si;     
   PROCESS_INFORMATION pi;

   // set the size of the structures
   ZeroMemory( &si, sizeof(si) );
   si.cb = sizeof(si);
   ZeroMemory( &pi, sizeof(pi) );

  // start the program up
  CreateProcess( lpApplicationName,   // the path
    args,        // Command line
    NULL,           // Process handle not inheritable
    NULL,           // Thread handle not inheritable
    FALSE,          // Set handle inheritance to FALSE
    0,              // No creation flags
    NULL,           // Use parent's environment block
    NULL,           // Use parent's starting directory 
    &si,            // Pointer to STARTUPINFO structure
    &pi             // Pointer to PROCESS_INFORMATION structure (removed extra parentheses)
    );
    // Close process and thread handles. 
    CloseHandle( pi.hProcess );
    CloseHandle( pi.hThread );
}

//F-cja uruchamiająca program z parametrami string sciezka/nazwa_aplikacji, string argumenty
void Startup(string appNameOrPath, string args=NULL)
{
	//zmiana stringa na const char*
	const char * cPath = appNameOrPath.c_str();
	//zamiana stringu na char*
	char * cArgs = new char[args.length() + 1];
	strcpy(cArgs, args.c_str());
	
		
   	// additional information
   	STARTUPINFO si;     
   	PROCESS_INFORMATION pi;

   	// set the size of the structures
   	ZeroMemory( &si, sizeof(si) );
   	si.cb = sizeof(si);
   	ZeroMemory( &pi, sizeof(pi) );

  	// start the program up
  	if(!CreateProcess( cPath,   // the path
    	cArgs,        // Command line
    	NULL,           // Process handle not inheritable
    	NULL,           // Thread handle not inheritable
    	FALSE,          // Set handle inheritance to FALSE
    	0,              // No creation flags
    	NULL,           // Use parent's environment block
    	NULL,           // Use parent's starting directory 
    	&si,            // Pointer to STARTUPINFO structure
    	&pi             // Pointer to PROCESS_INFORMATION structure (removed extra parentheses)
    ))
    	cout<<"Nie udalo sie uruchomic "<<exec<<endl;
    WaitForSingleObject( pi.hProcess, INFINITE );
   
    DWORD exitcode;
    if( !GetExitCodeProcess( pi.hProcess, & exitcode ) ) 
	{
        printf( "Nie udalo sie otrzymac komunikatu bledu (%ld)", GetLastError() );
        CloseHandle( pi.hProcess );
        CloseHandle( pi.hThread );
    }   
    else
    {	// Close process and thread handles. 
		cout<<"Instalacja "<<insName<<" zostala zakonczona"<<endl; 
    	CloseHandle( pi.hProcess );
    	CloseHandle( pi.hThread );
	}
    delete [] cArgs;
}

//F-cja zwracająca zawartość schowka jako string
string GetClip()
{
	string clipString;
	HANDLE clip;
	if (OpenClipboard(NULL)) 
	{
      clip = GetClipboardData(CF_TEXT);
      CloseClipboard();
    }
    clipString = (char*)clip;
    return clipString;
}

//ProgramyDoZainstalowania.cfg

//F-cja zbierająca dane z pliku ProgramyDoZainstalowania.cfg i przechowująca je na stosie
void GetInstaller(queue <string> & idInstallQueue)
{
	fstream installer;
	string temp;
    installer.open( "ProgramyDoZainstalowania.cfg", ios::in | ios::out );
    if( installer.good() == true )
    {
   	//pobieranie danych z pliku aż do jego końca
        while( !installer.eof() )
        {
            getline( installer, temp );
            //sprawdzenie czy wiersz nie jest komentarzem oraz czy linia nie jest pusta
            if((temp[0]!='#')&&(temp!=""))
			{		
			//usuwanie spacji
			SpaceEraser(temp);
			//wrzucenie do kolejki wiesza
			idInstallQueue.push(temp);
			}
        } 
        installer.close();
        returnGetInstaller=0;
    }
    else
	{
		returnGetInstaller=1;
	}
	
}

//F-cja sprawdzająca czy poprawnie wczytano dane do instalacji z pliku ProgramyDoZainstalowania.cfg
void CheckInstaller(int returnGetInstaller)
{
	if(returnGetInstaller==0)
		cout<<"Pomyslnie wczytano plik ProgramyDoZainstalowania.cfg"<<endl;
	else
		cout<<"Blad pliku ProgramyDoZainstalowania.cfg"<<endl;
}

//ListaProgramow.cfg

//F-cja zamieniająca stringa na dwa parametry idPackage & idComponent
void GetPackComp(string line)
{
	string temp="";
	for(int i=0;i<line.length();i++)
	{
		if(line[i]!=';')
			temp=temp+line[i];
		else
		{
			idPackage = temp;
			temp="";
		}
	}
	idComponent=temp;
}

//F-cja wczytująca: Nazwę_Komponentu, Lokalizację, Plik_wykonalny, Argumenty
bool SetNamePathExecArgs(string line)
{
	//zmienna typu int sprawdzająca czy zmieniono już zawartość zmiennych
	int isSet=0;
	//implementacja gdy idComponent=0 - wtedy ma wszystko wypisac z pakietu
	{/*
	if(idComponent==0)
	{
		
	}
	*/}
	//stworzenie tymczasowej zmiennej pomocniczej
	string temp="";
	bool bReturn;
	for(int i=0;i<line.length();i++)
	{
		if(line[i]!=';')
			temp=temp+line[i];
		else
		{
			switch (isSet)
			{
				case 0:
				{
					//sprawdzenie czy idPackage jest takie samo jakiego szukamy
					//jeżeli nie zwraca FALSE(kończy przeszukiwanie wiersza)
					//jeżeli tak - zwiększa isSet i kontynuuje sprawdzanie dalej
					if(idPackage==temp)
						{
							isSet++;
							temp="";
						}
					else
						return false;
					break;
				}
				case 1:
				{
					//sprawdzenie czy idComponent jest takie samo jakiego szukamy
					//jeżeli nie zwraca FALSE(kończy przeszukiwanie wiersza)
					//jeżeli tak - zwiększa isSet i zaczyna pobierać dane
					if(idComponent==temp)
						{
							isSet++;
							temp="";
						}
					else
						return false;
					break;
				}
				case 2:
				{
					insName=temp;
					temp="";
					isSet++;
					break;
				}
				case 3:
				{
					path=temp;
					temp="";
					isSet++;
					break;
				}	
				case 4:
				{
					exec=temp;
					temp="";
					isSet++;
					break;
				}
				case 5:
				{
					args=temp;
					return true;
				}			
			}
		}
	}
}

//F-cja otwierająca plik ListaProgramow.cfg i uruchamiający wyszukiwanie
bool FindCFG()
{
	bool isFound=false;
	fstream cfg;
	string temp;
	
    cfg.open( "ListaProgramow.cfg", ios::in | ios::out );
    if( cfg.good() == true )
    {
   	//pobieranie danych z pliku aż do jego końca
        while( !cfg.eof() )
        {
            getline( cfg, temp );
            //sprawdzenie czy wiersz nie jest komentarzem oraz czy linia nie jest pusta
            if((temp[0]!='#')&&(temp!=""))
			{
				isFound=SetNamePathExecArgs(temp);
				if(isFound)
					return true;
			}
        } 
        cfg.close();
        returnFindCFG=0;
        return false;
    }
    else
	{
		returnFindCFG=1;
		return false;
	}
}

bool FindDelete()
{
	string delUp="DELETEFILES";
	string delDown="deletefiles";
	if((args=="")&&(path=="")&&(exec==""))
	{
		for(int i=0;i<delUp.length();i++)
		{
			if((delUp[i]!=insName[i])&&(delDown[i]!=insName[i]))
				return false;
		}
	}
	else
		return false;
	return true;
}

bool FindAsuser()
{
	string asUp="$ASUSER";
	string asDown="$asuser";
	for(int i=0;i<asUp.length();i++)
	{
		if((asUp[i]!=exec[i])&&(asDown[i]!=exec[i]))
			return false;
	}
	return true;
}

//F-cja usuwająca pliki z lokalizacji C:\\Atena\\PlikiInstalacyjne
void DeleteFiles()
{
	//stworzenie polecenia usuwania : 
	//komenda + lokalizacja_do_usunięcia+*.*
	string delPath="del /Q "+computerPath+"*.*";
	char yesOrNo;
	//zmiana stringa na char*
	const char * cPath = delPath.c_str();
	cout<<endl<<"Czy chcesz usunac pliki instalacyjne (T/N)?";
	cin>>yesOrNo;
	if((yesOrNo=='y')||(yesOrNo=='Y')||(yesOrNo=='t')||(yesOrNo=='T'))
	{
		system(cPath);//wykonanie polecenia
		cout<<spacer<<endl<<"Pliki zostaly usuniete"<<endl<<spacer<<endl;
	}
	else 
	{
		cout<<spacer<<endl<<"Pliki nie zostaly usuniete"<<endl<<spacer<<endl;
	}
}

//F-cja kopiująca pliki instalacyjne
bool CopyFiles()
{
	//stworzenie polecenia kopiowania : 
	//komenda + lokalizacja_do_przekopiowania + lokalizacja_do_której_ma_przekopiować
	string copyPath="xcopy \""+path+"\" \""+computerPath+"\" /Y";
	//zmiana stringa na char*
	const char * cPath = copyPath.c_str();
	//wykonanie polecenia
	system(cPath);
	return true;
}

//F-cja znajdująca w argumentach REGEDIT - dodawanie usuwanie regów
bool FindRegedit()
{
	string regeditUp="REGEDIT";
	string regeditDown="regedit";
	if(args.length()>7)
	{
		for(int i=0;i<7;i++)
		{
			//cout<<regeditUp[i]<<" "<<regeditDown[i]<<" "<<args[i]<<endl;
			if((regeditUp[i]!=args[i])&&(regeditDown[i]!=args[i]))
				return false;
		}
	}
	else
		return false;
	return true;
}

//F-cja wywołująca cmd z argumentami w lokalizacji instalacji
void CmdExec()
{
	string newPath="cd "+computerPath+" && "+args;
	const char * cmdPath = newPath.c_str();
	system(cmdPath);
	cout<<"cmd: "<<args<<endl;
}

//F-cja uruchamiająca kolejkę instalacji $$OPISAĆ!!!!!
void ExecInstaller(queue <string> & idInstallQueue)
{
	char yesOrNo;
	while(!idInstallQueue.empty())
	{
		GetPackComp(idInstallQueue.front());
		idInstallQueue.pop();
		if(FindCFG())
			{
				cout<<"Znaleziono: "<<insName<<" "<<path<<" "<<exec<<" "<<args<<endl;
				cout<<"Przechodze do kopiowania Komponentu "<<insName<<" do lokalizacji: "<<computerPath<<endl;
				if(CopyFiles())
				{
					cout<<"Przekopiowałem"<<endl<<spacer<<endl;
					if(exec!="")
					{
						if(FindAsuser())
						{
							cout<<"Uruchom : \""<<args<<"\" jako user (enter aby kontynuowac)"<<endl;
							system("pause");
							ShellExecute(NULL, "open", constPath, NULL, NULL, SW_SHOWDEFAULT);
						}
						else
							Startup(computerPath+exec,args);
					}
					else
					{
						if(FindDelete())
							DeleteFiles();							
						else if(FindRegedit())
						{
							cout<<"Czy chcesz wykonac : \""<<args<<"\" jako user (T/N)?"<<endl;
							cin>>yesOrNo;
							if((yesOrNo=='y')||(yesOrNo=='Y')||(yesOrNo=='t')||(yesOrNo=='T'))
								ShellExecute(NULL, "open", constPath, NULL, NULL, SW_SHOWDEFAULT);
							else
								CmdExec();			
						}
						else
							CmdExec();
					}
				}
			}
		else
			cout<<"Nie znaleziono paczki "<<idPackage<<" lub komponentu "<<idComponent<<endl<<spacer<<endl;
	}
}

int main(int argc, char* argv[]) 
{	
	queue <string> idInstallQueue;

	//TESTOWE PARAMETRY:
	{/*
	path="C:\\Program Files\\uvnc bvba\\UltraVnc\\vncviewer.exe";
	exec="";
	args=" -connect "+SpaceEraser(GetClip());
	idPackage=1;
	idComponent=1;
	*/}
	//sF-cja zbierająca dane z pliku ProgramyDoZainstalowania.cfg i przechowująca je na stosie
	GetInstaller(idInstallQueue);
	
	CheckInstaller(returnGetInstaller);
	
	ExecInstaller(idInstallQueue);
	
	DeleteFiles();
	
	//Uruchamianie programu w określonej lokalizacji
	//Startup(path,param);
	system("pause");
	return 0;
}
