---
title: Objective-C-Selektoren in Xamarin.iOS
description: In diesem Dokument wird erläutert, wie für die Interaktion mit Objective-C-Selektoren in c# wird. Es wird beschrieben, wie zum Aufrufen der Auswahlzeiger und technische Aspekte, die berücksichtigt werden müssen dabei.
ms.prod: xamarin
ms.assetid: A80904C4-6A89-389B-0487-057AFEB70989
ms.technology: xamarin-ios
author: bradumbaugh
ms.author: brumbaug
ms.date: 07/12/2017
ms.openlocfilehash: 3083770fd2874eca317585b6bf949f3efe56f879
ms.sourcegitcommit: aa9b9b203ab4cd6a6b4fd51e27d865e2abf582c1
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/30/2018
ms.locfileid: "39351729"
---
# <a name="objective-c-selectors-in-xamarinios"></a>Objective-C-Selektoren in Xamarin.iOS

Die Objective-C-Sprache basiert auf *Selektoren*. Auswahl wird eine Meldung, die auf ein Objekt gesendet werden kann oder ein *Klasse*. [Xamarin.iOS](~/ios/internals/api-design/index.md) Zuordnungen Selektoren in Instanzmethoden-Instanz und Klasse Selektoren auf statische Methoden.

Im Gegensatz zu normalen C#-Funktionen (und wie die C++-Memberfunktionen) Sie nicht direkt aufrufen, eine Auswahl mit [P/Invoke](http://www.mono-project.com/docs/advanced/pinvoke/).
(*Reserviert*: Sie könnten P/Invoke für nicht virtuelle C++-Memberfunktionen verwenden, in der Theorie, jedoch müssen Sie pro-Compiler die namenszerlegung, kümmern ist schwierig, die besser ignoriert.) Stattdessen Selektoren gesendet werden, auf ein Objective-C-Klasse oder Instanz mithilfe der [ `objc_msgSend` Funktion](http://developer.apple.com/mac/library/documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html#//apple_ref/c/func/objc_msgSend).

Möglicherweise [hilfreich, in der vorliegenden für Objective-C-messaging](http://developer.apple.com/iphone/library/documentation/cocoa/conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html) nützlich.

<a name="Example" />

## <a name="example"></a>Beispiel

Angenommen, Sie möchten zum Aufrufen der [-[NSString SizeWithFont:forWidth:lineBreakMode:]](http://developer.apple.com/iphone/library/documentation/UIKit/Reference/NSString_UIKit_Additions/Reference/Reference.html#//apple_ref/occ/instm/NSString/sizeWithFont:forWidth:lineBreakMode:) Selektor.
Die Deklaration (von Apple Dokumentation) ist:

```csharp
- (CGSize)sizeWithFont:(UIFont *)font forWidth:(CGFloat)width lineBreakMode:(UILineBreakMode)lineBreakMode
```

-  Der Rückgabetyp ist *CGSize* für die Unified API.
-  Die *Schriftart* -Parameter ist ein [UIFont](https://developer.xamarin.com/api/type/UIKit.UIFont/) (und ein Typ (indirekt) abgeleitet [NSObject](https://developer.xamarin.com/api/type/Foundation.NSObject/) ), und wird daher [System.IntPtr](xref:System.IntPtr) .
-  Die *Breite* -Parameter ein *CGFloat* , zugeordnet ist *Nfloat*.
-  Die *LineBreakMode* -Parameter ein *UILineBreakMode* , in Xamarin.iOS als bereits gebunden wurde die [UILineBreakMode Enumeration](https://developer.xamarin.com/api/type/UIKit.UILineBreakMode/) .


Alle Puzzleteile zusammensetzen, und wir möchten eine Objc_msgSend-Deklaration, die mit übereinstimmt:

```csharp
CGSize objc_msgSend(IntPtr target, IntPtr selector,
    IntPtr font, nfloat width, UILineBreakMode mode);
```

Wir benötigen, um sie zu deklarieren:

```csharp
[DllImport (Constants.ObjectiveCLibrary, EntryPoint="objc_msgSend")]
static extern CGSize cgsize_objc_msgSend_IntPtr_float_int (
    IntPtr target, IntPtr selector,
    IntPtr font,
    nfloat width,
    UILineBreakMode mode);
```

Nachdem deklariert, können wir ihn aufrufen, nachdem wir die entsprechenden Parameter haben:

```csharp
NSString      target = ...
Selector    selector = new Selector ("sizeWithFont:forWidth:lineBreakMode:");
UIFont          font = ...
nfloat          width = ...
UILineBreakMode mode = ...

CGSize size = cgsize_objc_msgSend_IntPtr_float_int(
    target.Handle, selector.Handle,
    font == null ? IntPtr.Zero : font.Handle,
    width,
    mode);
```

War der zurückgegebene Wert eine Struktur, die weniger als 8 Bytes groß war (z. B. die ältere `SizeF` vor dem Wechsel zu Unified-APIs verwendet) der obige Code würde auf dem Simulator aber abgestürzte auf dem Gerät ausgeführt haben. Um eine Auswahl aufrufen, die einen Wert kleiner als 8 Bits Größe, daher wir zurückgibt *auch* deklarieren müssen die `objc_msgSend_stret()` Funktion:

```csharp
[DllImport (MonoTouch.Constants.ObjectiveCLibrary, EntryPoint="objc_msgSend_stret")]
static extern void cgsize_objc_msgSend_stret_IntPtr_float_int (
    out CGSize retval,
    IntPtr target, IntPtr selector,
    IntPtr font,
    nfloat width,
    UILineBreakMode mode);
```

Aufruf würde dann werden:

```csharp
NSString      target = ...
Selector    selector = new Selector ("sizeWithFont:forWidth:lineBreakMode:");
UIFont          font = ...
nfloat          width = ...
UILineBreakMode mode = ...

CGSize size;

if (Runtime.Arch == Arch.SIMULATOR)
    size = cgsize_objc_msgSend_IntPtr_float_int(
        target.Handle, selector.Handle,
        font == null ? IntPtr.Zero : font.Handle,
        width,
        mode);
else
    cgsize_objc_msgSend_stret_IntPtr_float_int(
        out size,
        target.Handle, selector.Handle,
        font == null ? IntPtr.Zero: font.Handle,
        width,
        mode);
```


<a name="Invoking_a_Selector" />

## <a name="invoking-a-selector"></a>Aufrufen einer Auswahl

Aufrufen einer Auswahl umfasst drei Schritte:

1.  Das Ziel für die Auswahl zu erhalten.
1.  Ruft den Auswahlnamen.
1.  Rufen Sie objc_msgSend() mit den entsprechenden Argumenten ein.


<a name="Selector_Targets" />

### <a name="selector-targets"></a>Selector-Ziele

Ein Ziel für die Auswahl ist eine Instanz oder eine Objective-C-Klasse. Wenn das Ziel eine Instanz ist und stammt von einer Xamarin.iOS-Typ gebunden wird, verwenden Sie die [ObjCRuntime.INativeObject.Handle](https://developer.xamarin.com/api/property/ObjCRuntime.INativeObject.Handle/) Eigenschaft.

Wenn das Ziel einer Klasse ist, verwenden Sie [ObjCRuntime.Class](https://developer.xamarin.com/api/type/ObjCRuntime.Class/) verwenden, um einen Verweis auf die Instanz der Klasse zu erhalten, klicken Sie dann die [Class.Handle](https://developer.xamarin.com/api/property/ObjCRuntime.Class.Handle/) Eigenschaft.


<a name="Selector_Names" />

### <a name="selector-names"></a>Auswahlnamen

Selector-Namen werden in der Apple Dokumentation aufgeführt. Z. B. die [UIKit NSString Erweiterungsmethoden](http://developer.apple.com/iphone/library/documentation/UIKit/Reference/NSString_UIKit_Additions/Reference/Reference.html) enthalten [SizeWithFont:](http://developer.apple.com/iphone/library/documentation/UIKit/Reference/NSString_UIKit_Additions/Reference/Reference.html#//apple_ref/occ/instm/NSString/sizeWithFont:) und [SizeWithFont:forWidth:lineBreakMode:](http://developer.apple.com/iphone/library/documentation/UIKit/Reference/NSString_UIKit_Additions/Reference/Reference.html#//apple_ref/occ/instm/NSString/sizeWithFont:forWidth:lineBreakMode:). Die eingebetteten und nachfolgende Doppelpunkte sind wichtig, und sind Teil des Namens der Auswahl.

Wenn Sie einen Auswahlnamen haben, können Sie erstellen eine [ObjCRuntime.Selector](https://developer.xamarin.com/api/type/ObjCRuntime.Selector/) Instanz dafür.


<a name="Calling_objc_msgSend()" />

### <a name="calling-objcmsgsend"></a>Aufrufen von objc_msgSend()

 `objc_msgSend()` Dient zum Senden einer Nachricht (Selektor) an ein Objekt. Diese Gruppe von Funktionen wird über mindestens zwei erforderliche Argumente: das Selector-Ziel (eine Instanz oder eine Klasse behandelt), die Auswahl selbst und dann beliebige Argumente, die für die bestimmte-Auswahl erforderlich. Die Instanz und der Auswahlzeiger-Argumente müssen sein `System.IntPtr`, und alle übrigen Argumente müssen den Typ der Selektor, z. B. erwartet mit einer `nint` für eine `int`, oder ein `System.IntPtr` für alle `NSObject`-abgeleiteten Typen. Verwenden der [NSObject.Handle](https://developer.xamarin.com/api/property/Foundation.NSObject.Handle/) -Eigenschaft zum Abrufen einer `IntPtr` für eine Instanz der Objective-C-Typ.

Leider ist mehr als eine `objc_msgSend()` Funktion.

Verwendung [ `objc_msgSend_stret()` ](http://developer.apple.com/mac/library/documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html#//apple_ref/c/func/objc_msgSend_stret) für Selektoren, die eine Struktur zurückgeben.
Enthält alle Rückgabetypen, die "interessant", klicken Sie auf ARM dies halber *nicht* Enumeration oder C# integrierten Typen (Char, short, Int, long, float, double). Auf X86 (der Simulator) muss dies für alle Strukturen, die größer als 8 Bytes Groß verwendet werden. (CGSize – verwendet, in dem Beispiel oben – 8 Bytes genau, und daher nicht `objc_msgSend_stret()` im Simulator.)

Verwendung [ `objc_msgSend_fpret()` ](http://developer.apple.com/mac/library/documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html#//apple_ref/c/func/objc_msgSend_fpret) für Selektoren, die eine Gleitkommazahl zurückgeben Wert nur X86 zeigen. Diese Funktion muss nicht auf ARM verwendet werden. Verwenden Sie stattdessen `objc_msgSend()`.

Die Main [objc_msgSend()](http://developer.apple.com/mac/library/documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html#//apple_ref/c/func/objc_msgSend) Funktion wird für alle anderen Selektoren verwendet.

Nachdem Sie die entschieden haben `objc_msgSend()` Funktion(en) aufrufen, müssen Sie (und möglicherweise mehr als 1 ist, z. B. für den Simulator und Gerät), können Sie einen normalen [ `[DllImport]` ](xref:System.Runtime.InteropServices.DllImportAttribute) Methode, um die Funktion für den späteren Aufruf deklarieren.

Eine Reihe von vorgefertigte `objc_msgSend()` Deklarationen finden Sie im [ `ObjCRuntime.Messaging` ](https://developer.xamarin.com/api/type/ObjCRuntime.Messaging/).


<a name="ugly" />

## <a name="the-ugly"></a>The Ugly

Objective-C verfügt über drei Arten von `objc_msgSend` Methoden: eine für den regulären aufrufen, eine für Aufrufe, die Gleitkommawerte (nur X86) zurückgeben und eine für Aufrufe, die Strukturwerte zurückgeben. Letztere enthält das Suffix `_stret` in `ObjCRuntime.Messaging`.

Wenn Sie eine Methode aufrufen, die bestimmte Strukturen zurückgibt (Regeln werden nachfolgend beschrieben), müssen Sie die Methode mit dem Rückgabewert als erster Parameter als ein falscher Wert müssen wie folgt aufrufen:

```csharp
// The following returns a PointF structure:
PointF ret;
Messaging.PointF_objc_msgSend_stret_PointF_IntPtr (out ret, this.Handle, selConvertPointFromWindow.Handle, point, window.Handle);
```

Dinge hier hässlichen und da die Regel für die Verwendung muss _Stret_ unterscheidet sich auf X86- und ARM, wenn Sie Ihre Bindungen auf den Simulator und dem Gerät arbeiten möchten müssen Sie Code hinzufügen, der folgendermaßen aussieht:

```csharp
if (Runtime.Arch == Arch.DEVICE){
    PointF ret;

    Messaging.PointF_objc_msgSend_stret_PointF_IntPtr (out ret, myHandle, selector.Handle);

    return ret;
} else
    return Messaging.PointF_objc_msgSend_PointF_IntPtr (myHandle, selector.Handle);
```

### <a name="using-the-objcmsgsendstret-method"></a>Verwenden die Objc\_MsgSend\_Stret-Methode

Die Regel für die Verwendung der [ `objc_msgSend_stret` ](http://developer.apple.com/mac/library/documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html#//apple_ref/c/func/objc_msgSend_stret) sind wie folgt für **ARM**:

-  Jeder Werttyp, der nicht auf eine Enumeration oder einer der Basistypen für eine Enumeration ist (Int, Byte, short, long, double, float).


Die Regel für die Verwendung der [ `objc_msgSend_stret` ](http://developer.apple.com/mac/library/documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html#//apple_ref/c/func/objc_msgSend_stret) sind wie folgt für **X86**:

-  Jeder Werttyp, der nicht auf eine Enumeration oder eines der Basistypen für eine Enumeration ist (Int, Byte, short, long, double, float), deren systemeigener Größe ist größer als 8 Bytes.


### <a name="creating-your-own-signatures"></a>Erstellen Ihre eigenen Signaturen.

Die folgenden [gist-Archiv](https://gist.github.com/rolfbjarne/981b778a99425a6e630c) können verwendet werden, um Ihre eigenen Signaturen, erstellen Sie bei Bedarf.



## <a name="related-links"></a>Verwandte Links

- [Beispiel für Selektoren](https://developer.xamarin.com/samples/mac-ios/Objective-C/Selectors/)
