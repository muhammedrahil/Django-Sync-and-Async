# Django-Async-Await

## Example 1
```python
import asyncio
import time

import aiohttp
import requests
from django.shortcuts import render

def example(request):

    starting_time = time.time()

    pokemon_data = []

    for num in range(1, 101):
        url = f"https://pokeapi.co/api/v2/pokemon/{num}"
        res = requests.get(url)
        pokemon = res.json()
        pokemon_data.append(pokemon["name"])

    count = len(pokemon_data)
    total_time = time.time() - starting_time

    return render(
        request,
        "index.html",
        {"data": pokemon_data, "count": count, "time": total_time},
    )

data - ['bulbasaur', 'ivysaur', 'venusaur', 'charmander', 'charmeleon', 'charizard', 'squirtle', 'wartortle', 'blastoise', 'caterpie', 'metapod', 'butterfree', 'weedle', 'kakuna', 'beedrill', 'pidgey', 'pidgeotto', 'pidgeot', 'rattata', 'raticate', 'spearow', 'fearow', 'ekans', 'arbok', 'pikachu', 'raichu', 'sandshrew', 'sandslash', 'nidoran-f', 'nidorina', 'nidoqueen', 'nidoran-m', 'nidorino', 'nidoking', 'clefairy', 'clefable', 'vulpix', 'ninetales', 'jigglypuff', 'wigglytuff', 'zubat', 'golbat', 'oddish', 'gloom', 'vileplume', 'paras', 'parasect', 'venonat', 'venomoth', 'diglett', 'dugtrio', 'meowth', 'persian', 'psyduck', 'golduck', 'mankey', 'primeape', 'growlithe', 'arcanine', 'poliwag', 'poliwhirl', 'poliwrath', 'abra', 'kadabra', 'alakazam', 'machop', 'machoke', 'machamp', 'bellsprout', 'weepinbell', 'victreebel', 'tentacool', 'tentacruel', 'geodude', 'graveler', 'golem', 'ponyta', 'rapidash', 'slowpoke', 'slowbro', 'magnemite', 'magneton', 'farfetchd', 'doduo', 'dodrio', 'seel', 'dewgong', 'grimer', 'muk', 'shellder', 'cloyster', 'gastly', 'haunter', 'gengar', 'onix', 'drowzee', 'hypno', 'krabby', 'kingler', 'voltorb']
 count - 100 
 time - 65.34059047698975

async def example(request):

    starting_time = time.time()

    pokemon_data = []

    async with aiohttp.ClientSession() as session:
        for num in range(1, 101):
            pokemon_url = f"https://pokeapi.co/api/v2/pokemon/{num}"
            async with session.get(pokemon_url) as res:
                pokemon = await res.json()
                pokemon_data.append(pokemon["name"])

    count = len(pokemon_data)
    total_time = time.time() - starting_time

    return render(
        request,
        "index.html",
        {"data": pokemon_data, "count": count, "time": total_time},
    )

data - ['bulbasaur', 'ivysaur', 'venusaur', 'charmander', 'charmeleon', 'charizard', 'squirtle', 'wartortle', 'blastoise', 'caterpie', 'metapod', 'butterfree', 'weedle', 'kakuna', 'beedrill', 'pidgey', 'pidgeotto', 'pidgeot', 'rattata', 'raticate', 'spearow', 'fearow', 'ekans', 'arbok', 'pikachu', 'raichu', 'sandshrew', 'sandslash', 'nidoran-f', 'nidorina', 'nidoqueen', 'nidoran-m', 'nidorino', 'nidoking', 'clefairy', 'clefable', 'vulpix', 'ninetales', 'jigglypuff', 'wigglytuff', 'zubat', 'golbat', 'oddish', 'gloom', 'vileplume', 'paras', 'parasect', 'venonat', 'venomoth', 'diglett', 'dugtrio', 'meowth', 'persian', 'psyduck', 'golduck', 'mankey', 'primeape', 'growlithe', 'arcanine', 'poliwag', 'poliwhirl', 'poliwrath', 'abra', 'kadabra', 'alakazam', 'machop', 'machoke', 'machamp', 'bellsprout', 'weepinbell', 'victreebel', 'tentacool', 'tentacruel', 'geodude', 'graveler', 'golem', 'ponyta', 'rapidash', 'slowpoke', 'slowbro', 'magnemite', 'magneton', 'farfetchd', 'doduo', 'dodrio', 'seel', 'dewgong', 'grimer', 'muk', 'shellder', 'cloyster', 'gastly', 'haunter', 'gengar', 'onix', 'drowzee', 'hypno', 'krabby', 'kingler', 'voltorb']
count - 100 
time - 20.74554753303528


async def get_pokemon(session, url):
    async with session.get(url) as res:
        pokemon_data = await res.json()
        return pokemon_data


async def example(request):

    starting_time = time.time()

    actions = []
    pokemon_data = []

    async with aiohttp.ClientSession() as session:
        for num in range(1, 101):
            url = f"https://pokeapi.co/api/v2/pokemon/{num}"
            actions.append(asyncio.ensure_future(get_pokemon(session, url)))

        pokemon_res = await asyncio.gather(*actions)
        for data in pokemon_res:
            pokemon_data.append(data["name"])

    count = len(pokemon_data)
    total_time = time.time() - starting_time

    return render(
        request,
        "index.html",
        {"data": pokemon_data, "count": count, "time": total_time},
    )
count - 100 
time - 1.715609073638916 
data - ['bulbasaur', 'ivysaur', 'venusaur', 'charmander', 'charmeleon', 'charizard', 'squirtle', 'wartortle', 'blastoise', 'caterpie', 'metapod', 'butterfree', 'weedle', 'kakuna', 'beedrill', 'pidgey', 'pidgeotto', 'pidgeot', 'rattata', 'raticate', 'spearow', 'fearow', 'ekans', 'arbok', 'pikachu', 'raichu', 'sandshrew', 'sandslash', 'nidoran-f', 'nidorina', 'nidoqueen', 'nidoran-m', 'nidorino', 'nidoking', 'clefairy', 'clefable', 'vulpix', 'ninetales', 'jigglypuff', 'wigglytuff', 'zubat', 'golbat', 'oddish', 'gloom', 'vileplume', 'paras', 'parasect', 'venonat', 'venomoth', 'diglett', 'dugtrio', 'meowth', 'persian', 'psyduck', 'golduck', 'mankey', 'primeape', 'growlithe', 'arcanine', 'poliwag', 'poliwhirl', 'poliwrath', 'abra', 'kadabra', 'alakazam', 'machop', 'machoke', 'machamp', 'bellsprout', 'weepinbell', 'victreebel', 'tentacool', 'tentacruel', 'geodude', 'graveler', 'golem', 'ponyta', 'rapidash', 'slowpoke', 'slowbro', 'magnemite', 'magneton', 'farfetchd', 'doduo', 'dodrio', 'seel', 'dewgong', 'grimer', 'muk', 'shellder', 'cloyster', 'gastly', 'haunter', 'gengar', 'onix', 'drowzee', 'hypno', 'krabby', 'kingler', 'voltorb']



from django.http import HttpResponse
import time, asyncio
from asgiref.sync import sync_to_async

# helper funcs

def get_movies():
    print('prepare to get the movies...')
    time.sleep(2)
    # qs = Movie.objects.all()
    # print(qs)
    print('got all the movies!')

def get_stories():
    print('prepare to get the stories...')
    time.sleep(5)
    # qs = Story.objects.all()
    # print(qs)
    print('got all the stories!')

@sync_to_async
def get_movies_async():
    print('prepare to get the movies...')
    time.sleep(2)
    # qs = Movie.objects.all()
    # print(qs)
    print('got all the movies!')

@sync_to_async
def get_stories_async():
    print('prepare to get the stories...')
    time.sleep(5)
    # qs = Story.objects.all()
    # print(qs)
    print('got all the stories!')

# views

def home_view(request):
    return HttpResponse('Hello world')

def main_view(request):
    start_time = time.time()
    get_movies()
    get_stories()
    total = (time.time()-start_time)
    print('total: ', total)
    return HttpResponse('sync')
    # total:  7.0053699016571045

async def example(request):
    start_time = time.time()
    # task1 = asyncio.ensure_future(get_movies_async())
    # task2 = asyncio.ensure_future(get_stories_async())
    # await asyncio.wait([task1, task2])
    await asyncio.gather(get_movies_async(), get_stories_async())
    total = (time.time()-start_time)
    print('total: ', total)
    return HttpResponse('async')

    # total:  5.002896070480347
```
