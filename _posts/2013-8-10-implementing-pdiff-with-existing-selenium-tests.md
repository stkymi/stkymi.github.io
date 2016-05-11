---
layout: post
title: Implementing PDiff with Existing Selenium Tests
---

I first heard how PDiff (Perceptual Diff) was used for testing web applications from [Brett Slatkin](http://www.onebigfluke.com/) via his presentation of ["The Secret to Safe Continuous Deployment"](http://www.youtube.com/watch?v=UMnZiTL0tUc). I am very impressed at how simple the technique is yet it can provide great value. As a developer who spend lots of time on UI, none of the existing automated test methods can spare me the trouble of firing up my browser and checking whether I messed up anything. Sure there is Selenium, but even though the data, which is what commonly get asserted by Selenium, is correct, the layout can be completely wrong.

## Implementing

So I was happy about PDiff and implemented my own work flow on Teamcity, a continue integration server from Jetbrain. The reason why I did not use [some](https://github.com/bslatkin/dpxdt) [of](https://github.com/facebook/huxley) [the](https://github.com/BBC-News/wraith) existing solutions is that we already have a pretty good Selenium test coverage. Integrating PDiff with existing Selenium means whenever you add a new Selenium test, PDiff can be added with little extra work.

In the test base class, add an extra method that works like this. I am gonna use Ruby here but it should work more or less the same in other languages

    def pdiff(name)
      page.capture_page("path/to/screenshot/folder/#{name}.png")
      # Keep a list of screenshots
      File.open(path/to/screenshot/folder/list.txt, 'a') do |f|
        f.puts "#{name}.png"
      end
      # Publish artifacts, or you can add path/to/screenshot/folder/ as the artifact folder.
      puts "##teamcity[publishArtifacts 'path/to/screenshot/folder/#{name}']"
    end
{: .prettyprint .lang-ruby}

In your tests, simple add one line in the flow you are testing.

    # In your test
    ...	
    pdiff "homepage-for-logged-in-user"
    ...
{: .prettyprint .lang-ruby}

Now you need to actually write your PDiff test. I recommend to create a new type of test, instead putting this with existing Selenium tests, which I will explain later.

    require 'open-uri'
    require 'chunky_png'
    
    # You can get url of artifacts of last and second to last successful_build from Teamcity REST API.
    last_url = ...
    second_to_last_url = ...
    open("last_url/#{list.text}") do |f|
    f.each_line do |line|
        it "#{line} should not have any differences" do  
          last = ChunkyPNG::Image.from_io(open("last_url/#{line}", 'rb'))
          second_to_last = ChunkyPNG::Image.from_io(open("second_to_last_url/#{line}", 'rb'))
          differences = 0
          (0...last.dimension.width).each do |x|
            (0...last.dimension.height).each do |y|
              # Highlight the difference in red
              if last[x, y] != second_to_last[x, y]
                last[x, y] = ChunkyPNG.Color.rgb(255, 0, 0)
                differences += 1
              end
            end
          end
          if difference != 0
            last.save("path/to/screenshot/folder/#{line}")
            # Publish artifacts, or you can add path/to/screenshot/folder/ as the artifact folder.
            puts "##teamcity[publishArtifacts 'path/to/screenshot/folder/#{line}']"
            fail_with "#{line} has #{differences} px differences"
          end
        end
      end
    end
{: .prettyprint .lang-ruby}

Then, in Teamcity, set up a new build type "PDiff" and list your Selenium as its dependency, i.e. before PDiff starts, Selenium must be run first. Then set up the build trigger so that PDiff will run for every check in.

*Fair warning: The above code is used for illustration only. I never test the above code (I will probably do it later when I have time). My original implementation is in Scala. So do not be surprised when you find the code does not work. Please correct any mistakes if you spot any.*

## Thoughts

After implementing PDiff, it actually caught lots of unexpected changes in UI, which would be only discovered at a very late stage without PDiff. As the extra effort when writing tests is minimal, whenever a new Selenium test is added, PDiff tests will be added. Things goes well and everyone is happy, the end. Of course that's not the truth. The work flow is far from perfect and there are quite few things that worth noticing.

PDiff in essence is a regression test: it does not have a static set of screenshots to compare to. It compares the last successful build with second to last and tell you what are the changes due to your check in. If your UI evolves very quickly, then PDiff build will be red very often, which is not good as people will start to ignore the red builds. So it's better to add PDiff once the UI you are testing against does not change that often.

Quite often you will check in some changes that will produce difference and therefore make the PDiff fail. This should not be called "fail" as it is actually expected. But PDiff can never tell the difference. That's why I recommend not to put PDiff and Selenium in the same build - you do not want your Selenium tests to give too many false alarms. The way to make the build green in the work flow described above is to manually trigger a build without any changes. This will definitely pass as the two builds that PDiff is comaparing are the same version of the code. You should only do this after checking the output of PDiff and confirm there are no unexpected changes. This is essentially saying the developer acknowledges and accepts the change. This work flow sounds a little "crude", but it covers 90% of the case. If for your UI changes are critical, you might want to build a more complicated work flow that allows developers (or QA) to accept part of the changes and reject the others and sign off changes (hey, next weekend project).

The work flow above assumes you have *good* Selenium tests: you start up a server, populated test data, fake the environment (like date and time) and run your Selenium tests. As the test data is the same, any difference in screenshots should be caused by your code. If you do not have Selenium tests, you may be able to take screenshots against a staging server that has static data. If you do not have that either, then thing is tricky. You should not run the tests against production server: as the data will almost always change, your PDiff will be flaky: you will not know whether a PDiff failure is caused by code change or simply data change. Flaky tests are the worst kind of tests - too many false alarms and people will complain and soon ignore the test result.

To sum up, if you have existing Selenium tests, adding PDiff almost takes no time. And I can assure you it is worth the effort. However, if you cannot control the variable to produce a stable screenshot, make sure you can before you implement PDiff.

[Discuss on HN](https://news.ycombinator.com/item?id=6191782)