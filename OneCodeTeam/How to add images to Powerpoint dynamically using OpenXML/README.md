# How to add images to Powerpoint dynamically using OpenXML
## Requires
- Visual Studio 2012
## License
- Apache License, Version 2.0
## Technologies
- Office
- Office Development
## Topics
- PowerPoint
- OpenXml
## Updated
- 06/13/2013
## Description

<h1>How to add images to PowerPoint dynamically using Open XML (CSOpenXmlInsertImageToPPT)</h1>
<h2>Introduction</h2>
<p class="MsoNormal">The sample demonstrates how to add image to PowerPoint file automatically using Open XML SDK.</p>
<p class="MsoNormal">We first insert a new slide into a PowerPoint file(NOTE: the PowerPoint file should have at least one slide). Then, inserting an image into the new slide. If no error occurs, you can open the PowerPoint file, and you will see the image
 has been added into the last slide; otherwise, a message box will pop up to report the error.</p>
<h2>Building the Sample</h2>
<p class="MsoNormal">Before building the sample, please make sure you have installed
<a href="http://www.microsoft.com/en-us/download/details.aspx?id=5124">Open XML SDK 2.0 for Microsoft Office.</a></p>
<h2>Running the Sample</h2>
<p class="MsoNormal">Step1. Open &quot;CSOpenXmlInsertImageToPPT.sln&quot; file and then click F5 to run the project. You will see the following Window Form:</p>
<p class="MsoNormal"><span style=""><img src="84369-image.png" alt="" width="470" height="275" align="middle">
</span></p>
<p class="MsoNormal">Step2. Click &quot;Open&quot; button to select an existing PowerPoint file that must have at least a slide.</p>
<p class="MsoNormal">Step3. Click &quot;Select&quot; button to select an existing image on your machine.</p>
<p class="MsoNormal">Step4. Click &quot;Insert&quot; button to insert the image to the PowerPoint file. If Inserting image is successful, you will see a message:</p>
<p class="MsoNormal"><span style=""><img src="84370-image.png" alt="" width="551" height="338" align="middle">
</span></p>
<p class="MsoNormal">Step5. Open source PowerPoint file and check whether the image has been added into the new slide.
</p>
<p class="MsoNormal"><span style=""><img src="84371-image.png" alt="" width="576" height="466" align="middle">
</span></p>
<h2>Using the Code</h2>
<p class="MsoNormal">Step1. Create Windows Forms Application project via Visual Studio</p>
<p class="MsoNormal">Step2. Add the Open Xml references to the project. To reference the Open XML, right click the project and click &quot;Add Reference…&quot; button. In the Add Reference dialog, navigate to the .Net tab, find DocumentFormat.OpenXml and
 WindowsBase, and then click &quot;Ok&quot; button. </p>
<p class="MsoNormal">Step3. Import Open XML namespace in class.</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span>
</div>
<span class="hidden">csharp</span>

<pre id="codePreview" class="csharp">
using DocumentFormat.OpenXml;
using DocumentFormat.OpenXml.Drawing;
using DocumentFormat.OpenXml.Office2010.Drawing;
using DocumentFormat.OpenXml.Packaging;
using DocumentFormat.OpenXml.Presentation;
using A = DocumentFormat.OpenXml.Drawing;
using P = DocumentFormat.OpenXml.Presentation;

</pre>
</div>
</div>
<div class="endscriptcode">&nbsp;</div>
<p class="MsoNormal">Step4. Insert a new slide into PowerPoint</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span>
</div>
<span class="hidden">csharp</span>

<pre id="codePreview" class="csharp">
/// &lt;summary&gt;
       /// Insert a new Slide into PowerPoint
       /// &lt;/summary&gt;
       /// &lt;param name=&quot;presentationPart&quot;&gt;Presentation Part&lt;/param&gt;
       /// &lt;param name=&quot;layoutName&quot;&gt;Layout of the new Slide&lt;/param&gt;
       /// &lt;returns&gt;Slide Instance&lt;/returns&gt;
       public Slide InsertSlide(PresentationPart presentationPart, string layoutName)
       {
           UInt32 slideId = 256U;


           // Get the Slide Id collection of the presentation document
           var slideIdList = presentationPart.Presentation.SlideIdList;


           if (slideIdList == null)
           {
               throw new NullReferenceException(&quot;The number of slide is empty, please select a ppt with a slide at least again&quot;);
           }


           slideId &#43;= Convert.ToUInt32(slideIdList.Count());


           // Creates an Slide instance and adds its children.
           Slide slide = new Slide(new CommonSlideData(new ShapeTree()));


           SlidePart slidePart = presentationPart.AddNewPart&lt;SlidePart&gt;();
           slide.Save(slidePart);
           
           // Get SlideMasterPart and SlideLayoutPart from the existing Presentation Part
           SlideMasterPart slideMasterPart = presentationPart.SlideMasterParts.First();
           SlideLayoutPart slideLayoutPart = slideMasterPart.SlideLayoutParts.SingleOrDefault
               (sl =&gt; sl.SlideLayout.CommonSlideData.Name.Value.Equals(layoutName, StringComparison.OrdinalIgnoreCase));
           if (slideLayoutPart == null)
           {
               throw new Exception(&quot;The slide layout &quot; &#43; layoutName &#43; &quot; is not found&quot;);
           }


           slidePart.AddPart&lt;SlideLayoutPart&gt;(slideLayoutPart);


           slidePart.Slide.CommonSlideData = (CommonSlideData)slideMasterPart.SlideLayoutParts.SingleOrDefault(
               sl =&gt; sl.SlideLayout.CommonSlideData.Name.Value.Equals(layoutName)).SlideLayout.CommonSlideData.Clone();


           // Create SlideId instance and Set property
           SlideId newSlideId = presentationPart.Presentation.SlideIdList.AppendChild&lt;SlideId&gt;(new SlideId());
           newSlideId.Id = slideId;
           newSlideId.RelationshipId = presentationPart.GetIdOfPart(slidePart);


           return GetSlideByRelationShipId(presentationPart, newSlideId.RelationshipId);
       }
       
       /// &lt;summary&gt;
       /// Get Slide By RelationShip ID
       /// &lt;/summary&gt;
       /// &lt;param name=&quot;presentationPart&quot;&gt;Presentation Part&lt;/param&gt;
       /// &lt;param name=&quot;relationshipId&quot;&gt;Relationship ID&lt;/param&gt;
       /// &lt;returns&gt;Slide Object&lt;/returns&gt;
       private static Slide GetSlideByRelationShipId(PresentationPart presentationPart, StringValue relationshipId)
       {
           // Get Slide object by Relationship ID
           SlidePart slidePart = presentationPart.GetPartById(relationshipId) as SlidePart;
           if (slidePart != null)
           {
               return slidePart.Slide;
           }
           else
           {
               return null;
           }
       }

</pre>
</div>
</div>
<div class="endscriptcode">&nbsp;</div>
<p class="MsoNormal">Step5. Insert image into the added new slide</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span>
</div>
<span class="hidden">csharp</span>

<pre id="codePreview" class="csharp">
/// &lt;summary&gt;
     /// Insert Image into Slide
     /// &lt;/summary&gt;
     /// &lt;param name=&quot;filePath&quot;&gt;PowerPoint Path&lt;/param&gt;
     /// &lt;param name=&quot;imagePath&quot;&gt;Image Path&lt;/param&gt;
     /// &lt;param name=&quot;imageExt&quot;&gt;Image Extension&lt;/param&gt;
     public void InsertImageInLastSlide(Slide slide, string imagePath, string imageExt)
     {
         // Creates an Picture instance and adds its children.
         P.Picture picture = new P.Picture();
         string embedId = string.Empty;
         embedId = &quot;rId&quot; &#43; (slide.Elements().Count() &#43; 915).ToString();
         P.NonVisualPictureProperties nonVisualPictureProperties = new P.NonVisualPictureProperties(
             new P.NonVisualDrawingProperties() { Id = (UInt32Value)4U, Name = &quot;Picture 5&quot; },
             new P.NonVisualPictureDrawingProperties(new A.PictureLocks() { NoChangeAspect = true }),
             new ApplicationNonVisualDrawingProperties());


         P.BlipFill blipFill = new P.BlipFill();
         Blip blip = new Blip() { Embed = embedId };


         // Creates an BlipExtensionList instance and adds its children
         BlipExtensionList blipExtensionList = new BlipExtensionList();
         BlipExtension blipExtension = new BlipExtension() { Uri = &quot;{28A0092B-C50C-407E-A947-70E740481C1C}&quot; };


         UseLocalDpi useLocalDpi = new UseLocalDpi() { Val = false };
         useLocalDpi.AddNamespaceDeclaration(&quot;a14&quot;,
             &quot;http://schemas.microsoft.com/office/drawing/2010/main&quot;);


         blipExtension.Append(useLocalDpi);
         blipExtensionList.Append(blipExtension);
         blip.Append(blipExtensionList);


         Stretch stretch = new Stretch();
         FillRectangle fillRectangle = new FillRectangle();
         stretch.Append(fillRectangle);


         blipFill.Append(blip);
         blipFill.Append(stretch);


         // Creates an ShapeProperties instance and adds its children.
         P.ShapeProperties shapeProperties = new P.ShapeProperties();


         A.Transform2D transform2D = new A.Transform2D();
         A.Offset offset = new A.Offset() { X = 457200L, Y = 1524000L };
         A.Extents extents = new A.Extents() { Cx = 8229600L, Cy = 5029200L };


         transform2D.Append(offset);
         transform2D.Append(extents);


         A.PresetGeometry presetGeometry = new A.PresetGeometry() { Preset = A.ShapeTypeValues.Rectangle };
         A.AdjustValueList adjustValueList = new A.AdjustValueList();


         presetGeometry.Append(adjustValueList);


         shapeProperties.Append(transform2D);
         shapeProperties.Append(presetGeometry);


         picture.Append(nonVisualPictureProperties);
         picture.Append(blipFill);
         picture.Append(shapeProperties);


         slide.CommonSlideData.ShapeTree.AppendChild(picture);


         // Generates content of imagePart.
         ImagePart imagePart = slide.SlidePart.AddNewPart&lt;ImagePart&gt;(imageExt, embedId);
         FileStream fileStream = new FileStream(imagePath, FileMode.Open);
         imagePart.FeedData(fileStream);
         fileStream.Close();
     }

</pre>
</div>
</div>
<div class="endscriptcode">&nbsp;</div>
<p class="MsoNormal">Step6. Handle the events of the button in Main Form</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span>
</div>
<span class="hidden">csharp</span>

<pre id="codePreview" class="csharp">
/// &lt;summary&gt;
        /// Handle client event of Insert button 
        /// &lt;/summary&gt;
        /// &lt;param name=&quot;sender&quot;&gt;&lt;/param&gt;
        /// &lt;param name=&quot;e&quot;&gt;&lt;/param&gt;
        private void btnInsert_Click(object sender, EventArgs e)
        {
  string pptFilePath = txbPPtPath.Text.Trim();
  string imagefilePath = txbImagePath.Text.Trim();
  string imageExt =Path.GetExtension(imagefilePath);
  if (imageExt.Equals(&quot;jpg&quot;, StringComparison.OrdinalIgnoreCase))
  {
      imageExt = &quot;image/jpeg&quot;;
  }
  else
  {
      imageExt = &quot;image/png&quot;;
  }
  bool condition =string.IsNullOrEmpty(pptFilePath)
      ||!File.Exists(pptFilePath)
      ||string.IsNullOrEmpty(imagefilePath)
      ||!File.Exists(imagefilePath);
  if (condition)
  {
      MessageBox.Show(&quot;The PowerPoint or iamge file is invalid,Please select an existing file again&quot;, &quot;Error&quot;, MessageBoxButtons.OK, MessageBoxIcon.Warning);
      return;
  }


  try
  {
      using (PresentationDocument presentationDocument = PresentationDocument.Open(pptFilePath, true))
      {
          // Get the presentation Part of the presentation document
          PresentationPart presentationPart = presentationDocument.PresentationPart;
          Slide slide = new InsertImage().InsertSlide(presentationPart, &quot;Title Only&quot;);
          new InsertImage().InsertImageInLastSlide(slide, imagefilePath, imageExt);
          slide.Save();
          presentationDocument.PresentationPart.Presentation.Save();
          MessageBox.Show(&quot;Insert Image Successfully,you can check with opening PowerPoint&quot;);
      }
  }
  catch (Exception ex)
  {
      MessageBox.Show(ex.Message, &quot;Error&quot;, MessageBoxButtons.OK, MessageBoxIcon.Error);
  }
        }


        /// &lt;summary&gt;
        /// Handle Click events of Open button
        /// &lt;/summary&gt;
        /// &lt;param name=&quot;sender&quot;&gt;&lt;/param&gt;
        /// &lt;param name=&quot;e&quot;&gt;&lt;/param&gt;
        private void btnOpenPPt_Click(object sender, EventArgs e)
        {
   using (OpenFileDialog dialog = new OpenFileDialog())
   {
      dialog.Filter = &quot;PowerPoint document (*.pptx)|*.pptx&quot;;
      dialog.InitialDirectory = Environment.CurrentDirectory;


      // Retore the directory before closing
      dialog.RestoreDirectory = true;
      if (dialog.ShowDialog() == DialogResult.OK)
      {
          this.txbPPtPath.Text = dialog.FileName;
      }
   }
        }


        /// &lt;summary&gt;
        /// Handle Click events of Select button
        /// &lt;/summary&gt;
        /// &lt;param name=&quot;sender&quot;&gt;&lt;/param&gt;
        /// &lt;param name=&quot;e&quot;&gt;&lt;/param&gt;
        private void btnSelectImg_Click(object sender, EventArgs e)
        {
  using (OpenFileDialog dialog = new OpenFileDialog())
  {
      dialog.Filter = &quot;Pictures(*.jpg;*.png)|*.jpg;*.png&quot;;
      dialog.InitialDirectory = Environment.CurrentDirectory;


      // Retore the directory before closing
      dialog.RestoreDirectory = true;
      if (dialog.ShowDialog() == DialogResult.OK)
      {
          this.txbImagePath.Text = dialog.FileName;
      }
  }
        }

</pre>
</div>
</div>
<div class="endscriptcode">&nbsp;</div>
<h2>More Information</h2>
<p class="MsoNormal">DocumentFormat.OpenXml.Presentation Namespace: </p>
<p class="MsoNormal"><a href="http://msdn.microsoft.com/en-us/library/office/documentformat.openxml.presentation(v=office.14).aspx">http://msdn.microsoft.com/en-us/library/office/documentformat.openxml.presentation(v=office.14).aspx</a>
</p>
<p class="MsoNormal">Open XML for Office developers: </p>
<p class="MsoNormal"><a href="http://msdn.microsoft.com/en-us/office/bb265236">http://msdn.microsoft.com/en-us/office/bb265236</a></p>
<p class="MsoNormal">Picture Class</p>
<p class="MsoNormal"><a href="http://msdn.microsoft.com/en-us/library/cc865819.aspx">http://msdn.microsoft.com/en-us/library/cc865819.aspx</a>
</p>
<p class="MsoNormal"></p>
<hr>
<div><a href="http://go.microsoft.com/?linkid=9759640" style="margin-top:3px"><img alt="" src="http://bit.ly/onecodelogo">
</a></div>
