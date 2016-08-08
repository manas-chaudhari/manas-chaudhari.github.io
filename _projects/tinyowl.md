---
layout: page
title: Software Development
date: "2014-06-01"

subtitles:
- "Company: TinyOwl Technology Pvt. Ltd."
- "June 2014 – present"
links:
- "<a href=\"https://play.google.com/store/apps/details?id=com.flutterbee.tinyowl&hl=en\">android</a>"
- "<a href=\"https://itunes.apple.com/in/app/runnr-food-ordering/id916154166?mt=8\">ios</a>"

---

{% include image.html src="tinyowl.jpeg" title="TinyOwl on iOS" align="right" size="small" %}
I joined TinyOwl as the first employee with the vision of solving ‘What should I eat?’. We started by building an Android app that allows you to order food from nearby restaurants. I was responsible for writing most of the non-UI modules such as database access, networking, data sync (The app worked offline in those days). I also participated in structuring backend APIs. When the Android app got stable, I built the iOS app and formed a team to maintain it.

Currently, as an engineering manager, I drive a full stack team of Android, iOS and backend (ROR) devs. I focus on increasing the productivity of the team. On a technical front, I create patterns to make development faster & bug-free. Some of my work:

- Use Functional Reactive Programming libraries to simplify pages with many dependent interactions
- Practise MVVM pattern to improve code reusability
  - In case of iOS, as there is no markup based data binding library, bindings are done programmatically using RxSwift
  - In Android, using RxJava + Android Data Binding allows creating very elegant view models
- Kotlin language which provides awesome nullability handling and allows writing extension method which is a nice language feature that allows to plugin multiple functionalities inside a class via interfaces (vs inheritance in which you can extend only one class)
- Phabricator/Arcanist: Phabricator allows to automate reviews by writing custom linters. This allowed devs to see errors before even creating a pull request. Allowing to compare multiple revisions of a pull request allows us to do incremental review (vs reading entire code again in other platforms)

In my experience, I have found that the challenge in developing apps lies in making development faster and optimizing performance. I strongly focus on solving these challenges. I recently started blogging about these techniques [here](/blog).
