# Lesson - Transforming Cloudinary images
## Learning Objectives
* Transforming Cloudinary images

## The Walkthrough

1. Get a copy of Lesson 12

2. Edit Cloudinary Class
    * Add the following lines of codes after the upload method:

```java
public String createUrl(String name) {
   return cloudinary.url().transformation( new Transformation().width(300)
           .height(300).crop("fill").radius(10)
   ).generate(name);
 }
```

    * After adding the code the Cloudinary class should look like this:

```java
import com.cloudinary.Cloudinary;
import com.cloudinary.Singleton;
import com.cloudinary.Transformation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import java.io.IOException;
import java.util.Map;

@Component
public class CloudinaryConfig {

  private Cloudinary cloudinary;

  @Autowired
  public CloudinaryConfig(@Value("${cloudinary.apikey}") String key,
                          @Value("${cloudinary.apisecret}") String secret,
                          @Value("${cloudinary.cloudname}") String cloud){
    cloudinary = Singleton.getCloudinary();
    cloudinary.config.cloudName=cloud;
    cloudinary.config.apiSecret=secret;
    cloudinary.config.apiKey=key;
  }

  public Map upload(Object file, Map options){
    try{
      return cloudinary.uploader().upload(file, options);
    } catch (IOException e) {
      e.printStackTrace();
      return null;
    }
  }

  public String createUrl(String name) {
    return cloudinary.url().transformation( new Transformation().width(300)
            .height(300).crop("fill").radius(10)
    ).generate(name);
  }
}
```

3. Edit  HomeController
    * Add the following lines of code inside processActor method.

```java
      String clodinaryImageId = (String) uploadResult.get("public_id");
      String transformedImage = cloudc.createUrl(clodinaryImageId);
      actor.setHeadshot(transformedImage);
```
   * HomeController should look like this now:

```java
import com.cloudinary.utils.ObjectUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.util.Map;

@Controller
public class HomeController {
  @Autowired
  ActorRepository actorRepository;

  @Autowired
  CloudinaryConfig cloudc;

  @RequestMapping("/")
  public String listActors(Model model){
    model.addAttribute("actors", actorRepository.findAll());
    return "list";
  }

  @GetMapping("/add")
  public String newActor(Model model){
    model.addAttribute("actor", new Actor());
    return "form";
  }

  @PostMapping("/add")
  public String processActor(@ModelAttribute Actor actor,
                             @RequestParam("file")MultipartFile file){
    if (file.isEmpty()){
      return "redirect:/add";
    }
    try {
      Map uploadResult =  cloudc.upload(file.getBytes(),
              ObjectUtils.asMap("resourcetype", "auto"));
      String clodinaryImageId = (String) uploadResult.get("public_id");
      String transformedImage = cloudc.createUrl(clodinaryImageId);
      actor.setHeadshot(transformedImage);
      actorRepository.save(actor);
    } catch (IOException e){
      e.printStackTrace();
      return "redirect:/add";
    }
    return "redirect:/";
  }
}

```

4. Run your application and open a browser go to http://localhost:8080/.


## What's Going On
In this example, we're adding transformation to the uploaded images.
All you need to do is upload the image to the Cloudianry server, and apply as
many styles as you would like to to transform the image.

## The Cloudinary Configuration Class

### public String createUrl()
This creates a Cloudinary URL 'preset' trasnformations. In this case, the width,
height, radius, and border can automatically be applied each time this method is called,
and a URL to the transformed image will be returned.

