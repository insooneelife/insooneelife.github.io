---
title: "[Unreal] SFML experience"
categories:
  - SFML
---

// SFML 사용 시 library

```c++
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
