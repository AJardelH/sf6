

## About the Project:

This project gets a number of different data for Street Fighter 6 from the [Street Fighter 6 rankings website](https://www.streetfighter.com/6/buckler/ranking/league) for use in data analysis to determine character usage and win rates.

## character_data:

The files in this folder look at the character usage for Street Fighter 6 characters, can be adjusted to look at specific leagues or ratings. 

- `get_sf6_ranks.py` loops through each rank and retrieves number of results as specified in the file. Output to csv
- `csv_join.py` combines csv's generated by `get_sf6_ranks.py`
- `masters_data.py` attempts to get **main** character for each player by sorting by rating and dropping duplicates (keeps highest rated character) and drops 1500 rated accounts (some may be active but many get Masters rank @1500 rating and do not play rank again)

- ## Visualisation:

The scripts above were used to gather a representative data set for each rank by returning the top 10,000 player / character combos in each rank (Rookie 1 through Masters). The total dataset contained 356,028 rows. The only division of the 36 not to have 10,000 players is Diamond 5. The assumption here is that if the player is good enough to achieve Diamond 5 they are good enough to get to Masters with enough playing. 

![image](https://github.com/AJardelH/SF6_Ranking_Data/assets/113073854/0fff3ba8-0701-4adf-927f-18c3d20aa81d)
*Example image of Tableau worksheet showing character popularity distrubution*

## [Look at the data on Tableau](https://public.tableau.com/authoring/StreetFighter6CharacterData/StreetFighter6CharacterPopularity)

## head_to_head:

This is an attempt to quantify the overall balance of the Street Fighter 6 roster at the highest level (Masters). 

The first step was to get a list of the top players. This was a matter of modifying the `get_sf6_ranks.py` to obtain what is the SF6 json as `short_id` and storing it in a local db to query as the table `players`.
This is a unique identifier of that player and can't be changed unlike their CFN and is also used for their Buckler Profile url. 

Over time we can then query the postgres db and pass each player into a request to get match data and loop through the ranked match pages for that player.
` url = f'https://www.streetfighter.com/6/buckler/profile/{short_id}/battlelog/rank?page={page}'`

From the request we get the following relevant data
- `'replay_id'` we can use this as a unique identifier to avoid counting matches twice
- `'player1_info.player.short_id'` gives us player 1 unique id 
- `'player1_info.master_rating'` gives us player 1's current Master Rating (at time of the match - need to check?)
- `'player1_info.character_name'` gives us which character they played
- `'player1_info.round_results'` #gives us a list of round results. This can be a 0 through 8~ which corresponds to an icon depending on how the round was won (example CA, OD, V, SA). Most importantly 0 is always L. We can use this to determine overall loser.
- `'player2_info.player.short_id',` the rest is as above but for player 2
- `'player2_info.master_rating',`
- `'player2_info.character_name',`
- `'player2_info.round_results'`

As we're not overly interested in how the match was won and only the overall winner we use the function `check_loss` which checks the list for two 0's or two losses to determine the winner and loser and drop the superfluous columns containing the lists.

`def check_loss(round_results):
    return round_results.count(0) >= 2`

We loop through and store these results in another table `matches` after some other transformations to make the data more readable

Due to the way the data is structured it became apparent that it was not going to be possible to accurately determine character win rates from this point. 
For example there may be 100 instances of Ryu winning as player1 in 150 mirror matches. When this was put in a matrix of row= p1 and col= p2 it would result in a 66% win for p1 when really it's a 50% win rate due to it being a mirror match.

To work around this we transform the data one last time by loading the query from `matches` into one dataframe, duplicating that dataframe and inverting the columns names so that p1 is p2 and p1_result is p2_result. These two dataframes are joined together and grouped and summed together to give us our head to head wins and losses for each character pair, regardless of if they were P1 or P2. 

This is then finally exported to a .csv for loading into Tableau. 

![image](https://github.com/AJardelH/SF6_Ranking_Data/assets/113073854/e3800a83-9f9d-417f-b302-563c2437b626)




