Step 1: Configure the MFC application to compile with CLR support

The best way to achieve interoperability between native C++ and managed .NET code is to compile the application as managed C++ rather than native C++. This is done by going to the Configuration Properties of the project. Under General there is an option "Common Language Runtime support". Set this to "Common Language Runtime Support /clr".

Step 2: Add the WPF assemblies to the project

Right-click on the project in the Solution Explorer and choose "References". Click "Add New Reference". Under the .NET tab, add WindowsBase, PresentationCore, PresentationFramework, and System. Make sure you Rebuild All after adding any references in order for them to get picked up.

Step 3: Set STAThreadAttribute on the MFC application

WPF requires that STAThreadAttribute be set on the main UI thread. Set this by going to Configuration Properties of the project. Under Linker->Advanced there is an option called "CLR Thread Attribute". Set this to "STA threading attribute".

Step 4: Create an instance of HwndSource to wrap the WPF component

System::Windows::Interop::HwndSource is a .NET class that handles the interaction between MFC and .NET components. Create one using the following syntax:

```C#
System::Windows::Interop::HwndSourceParameters^ sourceParams = gcnew     System::Windows::Interop::HwndSourceParameters("MyWindowName");
sourceParams->PositionX = x;
sourceParams->PositionY = y;
sourceParams->ParentWindow = System::IntPtr(hWndParent);
sourceParams->WindowStyle = WS_VISIBLE | WS_CHILD;
```

```C#
System::Windows::Interop::HwndSource^ source = gcnew System::Windows::Interop::HwndSource(*sourceParams);
source->SizeToContent = System::Windows::SizeToContent::WidthAndHeight;
```
Add an HWND member variable to the dialog class and then assign it like this: 

```C#
m_hWnd = (HWND) source->Handle.ToPointer();
```

The source object and the associated WPF content will remain in existence until you call ::DestroyWindow(m_hWnd).

Step 5: Add the WPF control to the HwndSource wrapper

```C#
System::Windows::Controls::WebBrowser^ browser = gcnew System::Windows::Controls::WebBrowser();
```

```C#
browser->Height = height;
browser->Width = width;
source->RootVisual = browser;
```
Step 6: Keep a reference to the WPF object

Since the browser variable will go out of scope after we exit the function doing the creation, we need to somehow hold a reference to it. Managed objects cannot be members of unmanaged objects but you can use a wrapper template called gcroot to get the job done.

Add a member variable to the dialog class:

```C#
#include <vcclr.h>
gcroot<System::Windows::Controls::WebBrowser^> m_webBrowser;
```
Then add the following line to the code in Step 5:

```C#
m_webBrowser = browser;
```

Now we can access properties and methods on the WPF component through m_webBrowser.