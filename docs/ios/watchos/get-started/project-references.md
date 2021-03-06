---
title: WatchOS Projektverweise in Xamarin
description: Dieses Dokument beschreibt die Beziehung zwischen einer iOS-app, eine Watch-app und eine Watch-app-Erweiterung. Es wird erläutert, Projektverweise und Paket Bezeichner.
ms.prod: xamarin
ms.assetid: C366E062-C33D-406A-B3FF-CBE82E5D1E7E
ms.technology: xamarin-ios
author: bradumbaugh
ms.author: brumbaug
ms.date: 09/13/2016
ms.openlocfilehash: 1bd950d0929beae7133b0eb8ef6b2a69bc116f50
ms.sourcegitcommit: ea1dc12a3c2d7322f234997daacbfdb6ad542507
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/05/2018
ms.locfileid: "34791487"
---
# <a name="watchos-project-references-in-xamarin"></a>WatchOS Projektverweise in Xamarin

_Erläuterung der Beziehung zwischen der iOS-app, eine Watch-app und eine Watch-Erweiterung._

Sind die drei Projekte in einer Projektmappe WatchOS *automatisch konfigurierte* um aufeinander verweisen, auf eine bestimmte Weise für WatchOS 3 apps erstellt und ordnungsgemäß gebündelt werden. Diese Projektverweise und Paket-ID-Einstellungen werden nachfolgend beschrieben, für den Verweis.

## <a name="project-references"></a>Projektverweise

Zeigen Sie Verweise durch Doppelklicken auf den Knoten Verweise für jedes Projekt an:

- **iPhone-app** Verweise **Watch-App**

![](project-references-images/catalog-reference1.png "iPhone-app verweist Watch-App")

- **Überwachen der App** Verweise **Watch-App-Erweiterung**

![](project-references-images/catalog-reference2.png "iPhone-app verweist Watch-App")


 - Die **Watch-App-Erweiterung** verweist nicht auf eine der anderen Projekte

![](project-references-images/catalog-reference3.png "Sehen Sie sich, dass die App-Erweiterung nicht auf andere Projekte verweist")



## <a name="bundle-identifiers"></a>Paket-IDs

Sie müssen auch sicherstellen, Ihre **Bündel-IDs** richtig sind.
Alle drei Projekte müssen die *gleichen* ID-Präfix mit den Erweiterungen der vordefinierte müssen zwei überwachen-Projekten `watchkitextension` und `watchkitapp`wie folgt (für die **WatchKitCatalog** Beispiel):

 - Xamarin.iOS Unified Projekt- `com.xamarin.WatchKitCatalog`

 - WatchKit Erweiterungsprojekt- `com.xamarin.WatchKitCatalog.watchkitextension`

 - Watch-App-Projekt- `com.xamarin.WatchKitCatalog.watchkitapp`

Stellen Sie außerdem sicher, dass diese **"Info.plist"** Einstellungen korrekt sind:

 - Des Watch-App-Projekts `WKCompanionAppBundleIdentifier` entspricht der übergeordnete Container/app-Paket-ID (ie. das Projekt, das auf dem iPhone ausgeführt wird);

 - Des Kit Watch-Erweiterung, Projekts **WKApp-Paket-ID** entspricht der Watch-App-Projekt-Paket-ID.

Sie können die IDs bearbeiten, durch Doppelklicken auf die **"Info.plist"** Datei in jedem Projekt angelegt.

Diese Abbildung ist die **Watch-Erweiterung** Datei "Info.plist", mit der **Watch-App** Bezeichner als auch:

# <a name="visual-studio-for-mactabvsmac"></a>[Visual Studio für Mac](#tab/vsmac)
    
![](project-references-images/infoplist-extension.png "Diesem Screenshot ist die Datei von der Watch-Erweiterung \"Info.plist\"")

# <a name="visual-studiotabvswin"></a>[Visual Studio](#tab/vswin)
    
![](project-references-images/infoplist-extension-vs.png "Diesem Screenshot ist die Datei von der Watch-Erweiterung \"Info.plist\"")

-----

Diese Abbildung ist die **Watch-App** Datei "Info.plist".
Die aktuelle **überwachen OS** Version ist 8.2, damit die **Bereitstellungsziel** für die Watch-App sollte **8.2**. Beachten Sie, dass wenn Sie Xcode 6.3 installiert haben, dieser Wert werden, bis 8.3 festgelegt kann-sie ändern sollten, 8.2.

![](project-references-images/infoplist-watchapp.png "Der Apple Watch-Datei \"Info.plist\"")

Das Bereitstellungsziel entweder für die Watch-App kann sich von der Watch-Erweiterung und iOS-App unterscheiden.

