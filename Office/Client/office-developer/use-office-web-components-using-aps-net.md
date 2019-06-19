---
title: How to use Office Web Components by an XML Web service created using ASP.NET
description: 
author: simonxjx
manager: willchen
audience: ITPro
ms.prod: office-perpetual-itpro
ms.topic: article
ms.author: v-six
---

# How to use the Office Web Components with XML data generated by an XML Web service created using ASP.NET

## Summary

This step-by-step article describes how to use ASP.NET to build a simple XML Web service that retrieves data from the Microsoft Access sample Northwind database and returns the data as XML to a client when the client calls a method of the service. On the client side, the data is presented with the Office PivotTable and Chart components. This article also demonstrates how to build a client for the XML Web service by using either Visual Basic .NET or Visual Basic 6.0.

### Create a simple XML Web service using ASP.NET

1. Start Visual Studio. NET.   
2. On the **File** menu, click **New** and then click **Project**. Under **Project types** click **Visual Basic Projects**, then click **ASP.NET Web Service** under    
3. ******Templates**. Form1 is created by default.   
4. Change the **Location** to **http://localhost/MyDataService** and click **OK**. The XML Web service project is created on the local computer with the name MyDataService. Class Service1, which inherits from System.Web.Services.WebService, is created by default in the Service1.asmx file.   
5. In Solution Explorer, right-click **Service1.asmx**, and then select **View Code**. This displays the code-behind file for the .asmx page.   
6. Add a reference to the Microsoft ActiveX Data Objects library. To do this, follow these steps:

   1. On the **Project** menu, click **Add Reference**.   
   2. Click the **COM** tab, select **Microsoft ActiveX Data Objects 2.7 Library**, click **Select**, and then click **OK**.   
   
7. Add the following method to Class Service1:
    ```cs
    <WebMethod()> Public Function GetResultsAsAdoXML() As String
        Dim myAdoRs As ADODB.Recordset
        Dim myAdoConnection As New ADODB.Connection()
        Dim mypersiststream As New ADODB.Stream()
    
    Dim myConnectionString As String = "Provider=Microsoft.Jet.OLEDB.4.0; Data Source=" & _
           "C:\Program Files\Microsoft Office\Office10\Samples\Northwind.mdb"
        Dim mySelect As String = "SELECT * from [Category Sales for 1997]"
    
    myAdoConnection.ConnectionString = myConnectionString
        myAdoConnection.Open()
        myAdoRs = myAdoConnection.Execute(mySelect)
        myAdoRs.Save(mypersiststream, ADODB.PersistFormatEnum.adPersistXML)
        Return mypersiststream.ReadText
    End Function
    ```

    > [!NOTE]
    > If you did not install Office XP to the default folder (C:\Program Files\Microsoft Office), modify the connection string in the code to reflect the correct path for your Office installation. Also, if you have installed Office 2003, you may have to change the path accordingly.

8. On the **Build** menu, click **Build Solution** to build the XML Web service.   
### Test Your XML Web Service

1. On the **Debug** menu, click **Start**. Microsoft Internet Explorer starts and browses to the Service1.asmx file.   
2. Click the **GetResultsAsAdoXML** hyperlink.    
3. Click **Invoke** to test the method call. A new window displays the results of the
Category Sales for 1997 query in the XML rowset schema format. The rowset XML document is enclosed in the string element.   
4. On the **Debug** menu in Visual Studio .NET, click **Stop Debugging**.   

### Build a Client for Your XML Web Service
 
This section describes how to create a client to consume the MyDataService XML Web service. You can create your client with either Visual Basic 6.0 or Visual Basic .NET.

For demonstration purposes, the steps describe how to build the test client on the same computer as the XML Web service. However, you can build your test clients on any computer that can browse to the Web Services Description Language (WSDL) file that the Web service exposes.

#### Use Visual Basic .NET

1. Create a new Visual Basic .NET Windows Application project. Form1 is created by default.   
2. Press CTRL+ALT+X to display the Toolbox.   
3. On the **Tools** menu, click **Customize Toolbox**. Select the following items in the components list, and then click **OK**:

   - Microsoft Office Chart 10.0   
   - Microsoft Office Data Source Control 10.0   
   - Microsoft Office PivotTable 10.0   
 Icons for the components appear in the Toolbox.

4. Add a Chart control, a PivotTable control, and a Data Source control to Form1. Size and position the form and controls as needed.   
5. Add a Button control to Form1, and then set the Text property of the control to Fill Data.   
6. Add a reference to the sample XML Web service. To do this, follow these steps:

   1. On the **Project** menu, click **Add Web Reference**. The **Add Web Reference** dialog box appears.   
   2. In the **Address** text box, type http://**localhost**/MyDataService/Service1.asmx?wsdl and press ENTER. The WSDL file is displayed in the dialog box.

    Note: If MyDataService does not reside on the local computer, replace **localhost** in the address with the name of the Web server on which MyDataService resides.   
   3. In the dialog box, click **Add Reference**.   
   
7. Double-click the Button control on Form1, and then replace the Button1_Click handler with the following code:
    ```vb
    Private Sub Button1_Click(ByVal sender As System.Object, ByVal e As System.EventArgs) Handles Button1.Click
    
    'Reference to the service.
        Dim objClient As New localhost.Service1()
    
    'Result obtained from the Service method.
        Dim strresult As String
    
    'The results will be stored on this disk file.
        Dim datafile As String
    
    datafile = System.IO.Directory.GetCurrentDirectory() + "\\mydata.xml"
    
    'Call the Web service method GetResultsAsAdoXML, which returns the recordset as XML.
        strresult = objClient.GetResultsAsAdoXML()
    
    Dim oxmldoc As New System.Xml.XmlDocument()
    
    'Load the XML recordset into the XML Document object.
        oxmldoc.LoadXml (strresult)
    
    'Save it to disk file.
        oxmldoc.Save (datafile)
    
    'Get data in the DataSource component.
        If Len(AxDataSourceControl1.ConnectionString) = 0 Then
            AxDataSourceControl1.ConnectionString = "provider=mspersist"
            AxDataSourceControl1.RecordsetDefs.AddNew( _
                  datafile, AxDataSourceControl1.Constants.dscCommandFile, "ChartData")
        End If
    
    'Get data in Chart component.
        Dim c
        Dim cht As OWC10.ChChart
        Dim ser As OWC10.ChSeries
        Dim ax As OWC10.ChAxis
    
    c = AxChartSpace1.Constants
        ' Clear the ChartSpace.
        AxChartSpace1.Clear()
    
    'DataSource to the Chart component.
        AxChartSpace1.DataSource = AxDataSourceControl1.DefaultRecordset.DataSource
    
    'Get the reference to Chart within the ChartSpace component.
        cht = AxChartSpace1.Charts(0)
        cht.Type = c.chChartTypeBarStacked
        cht.HasLegend = True
        cht.Legend.Position = c.chLegendPositionTop
        cht.HasTitle = True
        cht.Title.Caption = "Category Sales for 1997"
    
    'Set the Chart data.
        AxChartSpace1.SetData(OWC10.ChartDimensionsEnum.chDimCategories, 0, "CategoryName")
        AxChartSpace1.SetData(OWC10.ChartDimensionsEnum.chDimValues, 0, "CategorySales")
    
    ax = cht.Axes(c.chAxisPositionBottom)
    
    'Set the series attributes.
        ser = cht.SeriesCollection(0)
        ser.Name = "Category Sales"
        ser.Caption = ser.Name
        ser.Marker.Size = 4
    
    'Get Data in the PivotTable component.
        Dim pview As OWC10.PivotView
        AxPivotTable1.AutoFit = True
        AxPivotTable1.ConnectionString = "provider=mspersist"
        AxPivotTable1.CommandText = datafile
        pview = AxPivotTable1.ActiveView
        pview.AutoLayout()
        pview.FilterAxis.Label.Visible = False
        pview.RowAxis.Label.Visible = False
        pview.ColumnAxis.Label.Visible = False
        pview.TitleBar.Visible = False
    
    End Sub
    
    ```
    **Note** If the XML Web service is on a separate computer, declare the objClient variable as follows: 
    ```vb
     Dim objClient As New <servername>.Service1()
    ```
     Where **servername** is the name of the server on which the XML Web service resides.   
8. Note If you are using Office 2003, you may have to change the references accordingly.

9. Add the following code to the top of Form1.vb:
    ```vb
    Imports OWC10 = Microsoft.Office.Interop.OWC
    
    ```

10. On the **Debug** menu, click **Start** to build and run the client program. Form1 appears.   
11. Click **Fill Data** to present data in the Chart and PivotTable controls.   

#### Use Visual Basic 6.0
 
XML Web services communicate with clients by using Simple Object Access Protocol (SOAP) messages. If the client application is SOAP-aware, it can call the methods that are exposed by the XML Web service. The client computers need not have Visual Studio .NET installed to call methods on the XML Web service. The client should have the ability to frame XML Web service method calls as SOAP requests and receive the results of the method calls as SOAP responses. The SOAP Toolkit allows you to do this. 

For more information about the SOAP Toolkit, including download instructions, see the following Microsoft Developer Network (MSDN) Web site:
[SOAP Toolkit](https://msdn.microsoft.com/library/aa286526.aspx) 

To create a Visual Basic 6.0 test client to consume methods in the MyDataService XML Web service, follow these steps:

> [!NOTE]
> This test client requires that SOAP Toolkit 2.0 Service Pack 2 be installed on the computer. 

1. Start Visual Basic 6.0 and create a new Standard EXE project. Form1 is created by default.    
2. On the **Project** menu, click **References**. Add the following references to the project:

   - Microsoft Soap Type Library   
   - Microsoft XML, v3.0   
   
3. On the **Project** menu, click **Components**. Select **Microsoft Office XP Web Components**, and then click **OK**.   
4. Add the following Office Web Components to Form1:

   - Microsoft Office Chart 10.0   
   - Microsoft Office Data Source Control 10.0   
   - Microsoft Office PivotTable 10.0   
   
5. Add a CommandButton control to Form1 and set the Caption property of the button to Fill Data.   
6.  Double-click the CommandButton control and replace the Command1_Click handler with the following code:
    ```vb
    Private Sub Command1_Click()
      Dim osoapClient As MSSOAPLib.SoapClient
    
    'Initialize the SOAP client object.
      Set osoapClient = CreateObject("MSSOAP.SoapClient")
      osoapClient.mssoapinit "http://localhost/MyDataService/Service1.asmx?wsdl", "Service1", "Service1Soap"
    
    'DOM object to load the results.
      Dim oXMLDoc As New MSXML2.DOMDocument
      Dim strdirname As String
    
    'File to store XML recordset.
      strdirname = FileSystem.CurDir$ + "\mydata.xml"
    
    Dim strxmlrecordset As String
    
    'Call the WebService method.
      strxmlrecordset = osoapClient.GetResultsAsAdoXML()
    
    'Load the XML in the DOM document.
      oXMLDoc.loadXML strxmlrecordset
    
    'Save it to the disk file.
      oXMLDoc.Save (strdirname)
    
    'Get the data into the DataSourceControl1 component.
      If Len(DataSourceControl1.ConnectionString) = 0 Then
         DataSourceControl1.ConnectionString = "provider=mspersist"
         ' Add a RecordsetDef with name ChartData to the DataSourceControl1
         DataSourceControl1.RecordsetDefs.AddNew strdirname, DataSourceControl1.Constants.dscCommandFile, "ChartData"
      End If
    
    'Get the Data into the Chart component.
      Dim c
      Dim cht As ChChart
      Dim chart1 As ChChart
      Dim ser As ChSeries
      Dim ax As ChAxis
    
    Set c = ChartSpace1.Constants
      ChartSpace1.Clear
    
    'DataSource to the Chart component.
      ChartSpace1.DataSource = DataSourceControl1.DefaultRecordset.DataSource
    
    ' Draw the chart.
      Set cht = ChartSpace1.Charts(0)
      cht.Type = c.chChartTypeBarStacked
      cht.HasLegend = True
      cht.Legend.Position = c.chLegendPositionTop
      cht.HasTitle = True
      cht.Title.Caption = "Category Sales for 1997"
    
    'Set the chart data.
      ChartSpace1.SetData chDimCategories, 0, "CategoryName"
      ChartSpace1.SetData chDimValues, 0, "CategorySales"
    
    ' Set the tick label spacing depending on the number of points plotted.
      Set ax = cht.Axes(c.chAxisPositionBottom)
    
    'Set the series attributes.
      Set ser = cht.SeriesCollection(0)
      ser.Name = "Category Sales"
      ser.Caption = ser.Name
      ser.Marker.Size = 4
    
    'Get the data into the PivotTable.
      PivotTable1.ConnectionString = "provider=mspersist"
      PivotTable1.CommandText = strdirname
      PivotTable1.AutoFit = True
      Set pview = PivotTable1.ActiveView
      pview.AutoLayout
      pview.FilterAxis.Label.Visible = False
      pview.RowAxis.Label.Visible = False
      pview.ColumnAxis.Label.Visible = False
      pview.TitleBar.Visible = False
    End Sub
    
    ```
    **Note** If the XML Web service resides on a separate computer, replace **localhost** in the XML Web service address with the name of the server on which the XML Web service resides. 

7. Press F5 to build and run the program. Form1 appears.   
8. Click **Fill Data** to present data in the Chart and PivotTable components.   

## References

For more information about XML Web services that are created by using ASP.NET, click the following article number to view the article in the Microsoft Knowledge Base:

[301273](https://support.microsoft.com/help/301273) How to write a simple Web service by using Visual Basic .NET

For more information about XML Web services that are created by using ASP.NET, see the "Programming the Web with Web Services" topic in the Visual Studio .NET Help or the "ASP.NET Web Services and ASP.NET Web Service Clients" topic in the Microsoft .NET Framework Developer's Guide.

For additional information about Web service essentials and XML Web services that are created by using ASP.NET, visit the following Microsoft Developer Network (MSDN) Web site:

[https://msdn.microsoft.com/en-us/library/ba0z6a33.aspx](https://msdn.microsoft.com/library/ba0z6a33.aspx)

For additional information about Microsoft Office development with Visual Studio and Office XP Web Components, visit the following MSDN Web site:

[https://msdn.microsoft.com/en-us/library/aa188489(office.10).aspx](https://msdn.microsoft.com/library/aa188489%28office.10%29.aspx)