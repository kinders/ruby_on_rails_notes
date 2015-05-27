Devise is updated frequently.  New and removed features are often discussed on the [Platformatec blog](http://blog.plataformatec.com.br/tag/devise/) but may not always be explained in a step-by-step upgrade guide. So what do you need to check when you upgrade?

## Two key config files

Devise uses two primary configuration files for controlling its behavior. Generally these files are *not* changed during an upgrade, so it's up to you to review them and see if there are old features you need to remove or new features you would like to add. 

Compare your version to the version on GitHub to see what's different, and to learn what features you might want to investigate further. You can even do this before upgrading to get an idea whether there are changes that will impact your app.  Use an editor with a comparison feature (like Notepad++ on Windows) to do the comparison so you don't miss anything.

The GitHub links below are for the _master_ branch, which can be later than the released version. You may want to use GitHub history to retrieve the version corresponding to the update you are considering.

### devise.rb

This is the primary configuration file. Compare these two versions:

* In your app:  `config/initializers/devise.rb`
* Raw current source on GitHub:  [https://raw.github.com/plataformatec/devise/master/lib/generators/templates/devise.rb](https://raw.github.com/plataformatec/devise/master/lib/generators/templates/devise.rb)

**Note**  The GitHub version is actually a "generator" that is used to create the version on your computer during a new installation. The generator contains a few lines of Ruby code, e.g. to check your Rails version and to create a unique secret key. Do *not* copy this code into your app! Read the GitHub version carefully to determine what the corresponding text should be in your installation.

### devise.en.yml

This file contains the English translations of Devise's prompts and messages. Features that are added or removed from Devise often have corresponding changes to this file. Compare these two versions:

* In your app:  `config/locales/devise.en.yml`
* Raw current source on GitHub:  [https://raw.github.com/plataformatec/devise/master/config/locales/en.yml](https://raw.github.com/plataformatec/devise/master/config/locales/en.yml)