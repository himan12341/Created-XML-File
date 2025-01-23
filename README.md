# Created-XML-File
codeunit 50255 CreateXmlFile
{
    procedure CreateXmlAndDownload(): Text
    var
        XmlDoc: XmlDocument;
        XmlDec: XmlDeclaration;
        RootNode: XmlElement;
        ParentNode: XmlElement;
        Customer: Record Customer;
        //XMLText: XmlText;
        fieldCaption: Text;
        ChildNode: XmlElement;
        TempBlob: Codeunit "Temp Blob";
        Instr: InStream;
        OutStr: OutStream;
        ReadTxt: text;
        WriteTxt: Text;
    begin
        XmlDoc := XmlDocument.Create();
        XmlDec := XmlDeclaration.Create('1.0', 'UTF-8', '');
        XmlDoc.SetDeclaration(XmlDec);
        RootNode := XmlElement.Create('Table');
        XmlDoc.Add(RootNode);
        Customer.FindSet();
        repeat
            ParentNode := XmlElement.Create('Customer');
            RootNode.Add(ParentNode);
            fieldCaption := Customer.FieldCaption("No.");
            ChildNode := XmlElement.Create(fieldCaption);
            xml := XmlText.Create(Customer."No.");
            ChildNode.Add(xml);
            ParentNode.Add(ChildNode);

            fieldCaption := Customer.FieldCaption("Name");
            ChildNode := XmlElement.Create(fieldCaption);
            xml := XmlText.Create(Customer.Name);
            ChildNode.Add(xml);
            ParentNode.Add(ChildNode);

            fieldCaption := Customer.FieldCaption(Balance);
            ChildNode := XmlElement.Create(fieldCaption);
            xml := XmlText.Create(Format(Customer.Balance));
            ChildNode.Add(xml);
            ParentNode.Add(ChildNode);
        until Customer.Next() = 0;

        TempBlob.CreateInStream(Instr);
        TempBlob.CreateOutStream(OutStr);
        XmlDoc.WriteTo(OutStr);
        OutStr.WriteText(WriteTxt);
        Instr.ReadText(WriteTxt);
        ReadTxt := 'Custom.XML';
        DownloadFromStream(Instr, '', '', '', ReadTxt)
    end;

    procedure ImportXmlData()
    var
        FromFile: Text;
        Instr: InStream;
        xmldoc: XmlDocument;
        Tab: XmlElement;
        NodeList: XmlNodeList;
        NodeListSec: XmlNodeList;
        Node1: XmlNode;
        Node2: XmlNode;
        Element: XmlElement;
        Customer: Record Customer;
    begin
        if UploadIntoStream('UPLOAD XML FILE', '', '', FromFile, Instr) then
            XmlDocument.ReadFrom(Instr, xmldoc)
        else
            Error('Not an XML');

        if xmldoc.GetRoot(Tab) then
            NodeList := Tab.GetChildElements();

        foreach Node1 in NodeList do begin
            Customer.Init();
            Element := Node1.AsXmlElement();
            NodeListSec := Element.GetChildElements();

            foreach Node2 in NodeListSec do begin
                case Node2.AsXmlElement().Name of
                    'No.':
                        Customer.Validate(Customer."No.", Node2.AsXmlElement().InnerText);
                    'Name':
                        Customer.Validate(Customer.Name, Node2.AsXmlElement().InnerText);

                end
            end;
            Customer.Insert();
        end;
        Message('Process End');

    end;

    var
        xml: XmlText;
}
