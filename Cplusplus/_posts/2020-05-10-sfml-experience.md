---
title: "[c++] SFML experience"
categories:
  - SFML
---


```c++
// SFML 사용 시 library 
// sfml-window-d.lib
// sfml-system-d.lib
// sfml-graphics-d.lib를 사용해야 함

// visual studio project 경로를 수정해주어야 함
// project -> 속성 -> 구성속성 -> 디버깅 -> 작업 디렉터리
// $(SolutionDir)$(Platform)\$(Configuration)\  로 변경


sf::RenderWindow window(sf::VideoMode(960, 640), "name");

sf::Texture texture;
if (!texture.loadFromFile("temp.jpg"))
{
}
sf::Sprite sprite;
sprite.setTexture(texture);


// window 창
while (window.isOpen())
{
	sf::Event ev;

	while (window.pollEvent(ev))
	{
		cout << ev.type << endl;;
	}

	window.clear(sf::Color::White);
	window.draw(sprite);
	window.display();
}

return 0;
```
