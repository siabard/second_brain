Texture에 저수준으로 픽셀을 출력할 수 있다.
# `malloc`으로 버퍼 생성

가로 세로 200 픽셀의  버퍼를 만들고 RGBA 각 8비트값을 가지는 텍스쳐를 구성한다. 

```d
extern(C) {
  void* memset(void* s, int c, size_t n);
}

vod* buffer;
SDL_Texture* texture = SDL_CreateTexture(renderer, 
					   SDL_PIXELFORMAT_RGBA8888,
					   SDL_TEXTUREACCESS_TARGET,
					   cast(int)width,
					   cast(int)height);
    SDL_SetTextureBlendMode(this.scene_texture, SDL_BLENDMODE_BLEND);
buffer = malloc(200 * 200 * 4);
```

# 화면에 노출하도록 `buffer` 값을 설정한다.

픽셀 하나는 `uint32` (`ulong`)에 해당하니 한 바이트씩 RGBA 를 등록한다. 

```d
// 버퍼를 지운다
memset(buffer, 0, 200 * 200 * 4);

// pointer를 이용하여 랜덤하게 점을 찍는다.
for(uint y = 0; y < 200; y++) {
  for(uint x = 0; x < 200; x++) {
	auto r = x;
	auto g = y;
	auto b = (x + y) / 200;
	auto a = cast(uint)get_random(0, 255);
	auto pixel = (r << 24) + (g << 16) + (b << 16) + a;
       *cast(ulong*)(this.buffer + (y * 200 + x) * 4)  = pixel;
  }
}
```

그리고 텍스쳐를 버퍼로 업데이트한다. `SDL_UpdateTexture` 의 4번째 파라미터는 라인 하나의 길이이다. 가로가 200픽셀이고, RGBA 포맷이나 총 800바이트이다.

```d
auto rect = SDL_Rect(0, 0, 200, 200);
SDL_UpdateTexture(texture,
	&rect, 
	this.buffer,
	200 * 4);
  }
```

# 화면에 render 하면 된다.
```d
auto src_rect = SDL_Rect(0, 0, 
						 cast(int)this.game.wc.width,
						 cast(int)this.game.wc.height);
SDL_RenderCopyEx(renderer, 
				 texture,
				 &src_rect,
				 &src_rect,
				 0.0,
				 null,
				 SDL_FLIP_NONE);
```