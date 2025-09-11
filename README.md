# Jet-Res-Documentation
A summary of my time working at jet res. This includes a walkthrough through of the challenges I faced and solutions I achieved when working on both the mechanical and software development. Note this will not include code or a link to the repository due to the code falling under the companies IP

### About Jet Res
Jet Res is a small electonics manufacture manily focusing on low voulme surface mount pcb assembly. All of desginging, building ,testing and devlopning is all done in-house for both the harware and software requided to create a Picking and Placing machine for PBCs


# Software Development (Summer 0f 2025)
My role at jet res durning this time was to develope software that would take in images/video and output the location of a desiered object. This include both tracking and detection of very small compontes all the way up to larger components along with other common task need for Picking and placing. I developed multiple pipline for diffrent tasks that all employed both AI and analytical computer vision, this was then deplyed on to a cloud serevr for the pick and place machine to cal lthe pipeline as it wishes.

## Compontents and AI Vision

### AI Vision 
previouily for the use of ai in computer vision i tryed to use roboflow witch is an assortment of tools, models and software to alow for the creation of data , finetuning models and deploying models. Roboflow ended up beening very informative but ultimaily the wrong choice for this project due to conscerns with licence and the cost of useing the software, witch most of the funtunality wouldn't be used at this stage. 
After some reachserch I setteled on using the S.A.M 2 model from meta. It had incredible flexiabilty ,the abilty to track objects that are occluded and had an apache 2.0 license witch allows for free commercial distubution. I then buit up a easy to use class that alowed for simple instuctions to pass to said class and it handles all the data and formating S.A.M 2 requires and then return easily understood outputs.

## Detection of Components
Once componets have been picked up we then need to reculated where the componet is and it orination in order to compensatte for any unespetate movment in the picking process. this requiered many diffrent pipelines deppeding on this size of the Nozzle used witch in turn dependis on the size of the componet, I developed two of these piplines but due to having three nozzels a thrid will also need to be developed.

### Large Components
Large compents seemed to be an easier task to solve than others so a great starting point to test the S.A.M 2 model directy in an aplication it will be used for. From the imagage below you can see that the componet takes up most of the screen and only a small section of the nozzel mount is visble.

![00001](https://github.com/user-attachments/assets/7c07e08d-a2ff-4498-ba45-46670253dc0b)

We wanted to produce a mask of the object witch then will be used to find both the center and orintation of objectS with diffrent shapes. As you can see below the legs are ussaly picked up by the mask but with claerer prmoting we can attaully decied for each component wether or not to to include the legs as seen below. Including the legs in the mask can cause all sort of issuses , For example if a component has uneven legs the mask and therefor the center of the component will be shifted and biased towards the side with more legs.

legs arnt't grabed and the mask is perfect for finding the center of the component
<img width="640" height="480" alt="Large component_ mask" src="https://github.com/user-attachments/assets/d215e854-fbca-45d6-b519-71b7d3688a99" />

legs are included and this casue problems with the background beening detected as the component
<img width="640" height="480" alt="Large component_ mask legs included" src="https://github.com/user-attachments/assets/bca024f9-42c1-4e9e-be91-446c575344ab" />



### TINY COMPONENTS
For tiny compnent there was two main problems that need to be solved, the mask couldn't seprate the componet from the nozzle and I requiried one pixel where the componet is guaranteed to be present. The mask not separating the component was a major promblem, at first no matter how I promted S.A.M 2 sepration wouldnt occur. I resulted to using frame manupuilation istead, this is the process of taking raw shots from the camerea and then splcing frames in certian orders. using frame manupuilation along with cleaver promting the sepration was aachieved and work incredabily well. now in order to get the sepration I need one pixel that contains the component, this was a hard task as the compontent is so small we couldnt ganuratee that we could consistanly moved the nozzle into the frame with the uncearity required to reliable guarantee a pixel. so I reverted to using the mask that didnt seprate components from the nozzle as in order to pick the components the component must be covering the open section of the nozzel. This alowwed me to caculate the center of the nozzle from that first no seprating mask, this now gaurenterd a point where the componet must be present or it wouldnt be picked up. This took us from needing to place the nozzle in the frame with an uncearity based on the size of the component to the size of the nozzle making the process more relaible and possiable.


### Intail starting Image
![00001](https://github.com/user-attachments/assets/a1d66383-718c-40d7-9388-13273eeedb8f)

### Image with mask and bounding box annotated
<img width="1114" height="628" alt="sam2+pipeline output" src="https://github.com/user-attachments/assets/84fa5fd9-90ab-41f8-881e-3f1228edf7b5" />


# Sprockets ,Pads and Fussials

## Sprocekts
in order to pick up componets we need to know where they lie in relation to the PCB part of this is sprocket dection. As seen below there are components in a evenly spaced order along with a row of holes (the sprockets) also even spaced this is called a strip. The sprockets and componets have a knowen relation to each other so by knowing the sprockets you therefor know the posstion of the componets or vice versa. This is just as crucial as component orination and location dection seen above but this has to be tackled a diffrent way. First I had to rotate a larger image that contains either one or many strips to allow for missaliment in the strip placement. Then this is followed by cropping each indivial strip into its' own image to stop sprockets form other strips or the backgroud interfering.

Image to show that with rotation the sprockets are now more parrle with the bottom
<img width="1128" height="644" alt="Rotated Image verus Orignale" src="https://github.com/user-attachments/assets/2cdcd348-38bb-47ce-9d5b-4685f498a48b" />


From the cropped images I then use alogthim baseed computer vision to dector the sprockets, this sprocket detection worked really well but had a problem with lighting inconsistancy cuasing shadows in some sprockets causing incorrect centers to be found thus offseting the sprocket. Howerer due to shadows causing the issues this ment the alogthim based centers even though incorrect are always going to be within the sprockets, So therefor we have a knowen postion inside each sprocket (simailer to the tiny componet above).These incorrect sprockets are fine for larger componets as they will still be picked and can be correct by the above pipeline but for tiny componets they would not even get picked up with these sprockets. 

Show the geusee based on the alogrithm, some holes work well some are very missallined
<img width="1102" height="558" alt="Sprockets on image from algorithms" src="https://github.com/user-attachments/assets/655ba05d-f6bc-4e58-88c8-4e34030d783c" />


I then decied to test and implemed S.A.M 2 into the sprocket dection to refine the sprockets. By using the alogthim based centers as inputs into the S.A.M 2 model this resulted in shadows been ignored and more accurate srpockets beening found. Once i Have the points in the cropped roated image i then have to covert these points back into the uncropped and unroated image using matrix multiplication,subtracting and addition.

below is S.A.M 2 refining the sprockets, the top image is the larger uncropped un roated image with the sprockets overlayed on top, the bottom image is the mask that S.A.M 2 procdeuce for each sprocket.
<img width="1128" height="644" alt="Sprockets on image from AI" src="https://github.com/user-attachments/assets/082027bf-2eb5-415e-bf5a-762643d1eb7b" />
<img width="1000" height="324" alt="single strip sprocket masks" src="https://github.com/user-attachments/assets/239fcbd2-28e5-4c63-b707-0cd0161daea2" />



## Pads and Fudicals (tracking)
Pads are points on a PCB where componets will need be palced. Fudicalsvery dictancte marks on a pcb witch postion is knowen relative to the rest of the baord. Both of these fall into the same catagoary as they use the same pipeline due them both needing thier centers to be tracked. This was really easy to implenet as by now the S.A.M 2 class I created had alot of functionality. so all I had to do was take in imputs about what and how many objects wanted to be tracked. as seen below the points repsent the center of the mask and these point could be track even with fast movements and the object leaving the frame dure to S.A.M 2 occluation capabilties.

Notice that the points attually dont seem to be centered even through the masks capture the pads perfectly. This is due to how the points are drawn on to the image, this dos not effect the postion of center in any way only makes the visulation of said points look slight off.
<img width="1128" height="644" alt="Pads" src="https://github.com/user-attachments/assets/dd1670dd-08b8-462d-9497-410fa5c89281" />


# Conclution
by sussfully implmenting AI and alogthims I was able to create frameworks and piplines to help orientate, track and locate a set of both ambinoly sand regulary shaped componets,pads and parts. Using both reaserach papers, online threads and AI I was able to get a better undersatnding of S.A.M 2 allwoing me to manupuilate it stregths whilst minimsing it weakness. On top of this, due to my code intergating with a system development by others at the company, I learnt alot about colabrateive coding from pair programing to the vast feature-set of github.
I Greatly enjoyed my time at Jet Res durnig the summer. everyday had a challange and everyday I learnt something new with the added sence of accomplishment seening my code be intergrate into a work flow.

