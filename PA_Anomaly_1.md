```python
import requests
```


```python
def get_json_for_state(state):

    url = f'https://static01.nyt.com/elections-assets/2020/data/api/2020-11-03/state-page/{state}.json'
    r = requests.get(url)
    if r.status_code == 200:
        return r.json()
    else:
        print(
            f"Something went wrong with state {state}. I'm outputing the raw response")
        return r
```


```python
california_json = get_json_for_state('pennsylvania')['data']
```


```python
pres_race = list(filter(lambda x: x["race_name"] == "President", california_json['races']))[0]
```


```python
pres_race.keys()
```




    dict_keys(['race_id', 'race_slug', 'url', 'state_page_url', 'ap_polls_page', 'edison_exit_polls_page', 'race_type', 'election_type', 'election_date', 'runoff', 'race_name', 'office', 'officeid', 'nyt_race_description', 'race_rating', 'seat', 'seat_name', 'state_id', 'state_slug', 'state_name', 'state_nyt_abbrev', 'state_shape', 'party_id', 'uncontested', 'report', 'result', 'result_source', 'gain', 'lost_seat', 'votes', 'electoral_votes', 'absentee_votes', 'absentee_counties', 'absentee_count_progress', 'absentee_outstanding', 'absentee_max_ballots', 'provisional_outstanding', 'provisional_count_progress', 'poll_display', 'poll_countdown_display', 'poll_waiting_display', 'poll_time', 'poll_time_short', 'precincts_reporting', 'precincts_total', 'reporting_display', 'reporting_value', 'eevp', 'tot_exp_vote', 'eevp_source', 'eevp_value', 'eevp_display', 'county_data_source', 'incumbent_party', 'no_forecast', 'last_updated', 'candidates', 'has_incumbent', 'leader_margin_value', 'leader_margin_votes', 'leader_margin_display', 'leader_margin_name_display', 'leader_party_id', 'counties', 'precinct_metadata', 'votes2016', 'margin2016', 'clinton2016', 'trump2016', 'votes2012', 'margin2012', 'expectations_text', 'expectations_text_short', 'absentee_ballot_deadline', 'absentee_postmark_deadline', 'update_sentences', 'race_diff', 'winnerCalledTimestamp', 'model_metadata', 'timeseries'])




```python
pres_timeseries = pres_race["timeseries"]
pres_timeseries[0]
```




    {'vote_shares': {'trumpd': 0, 'bidenj': 0},
     'votes': 0,
     'eevp': 0,
     'eevp_source': 'edison',
     'timestamp': '2020-11-04T09:25:23Z'}




```python
def print_discrepancy(before_id):
    before_update = pres_timeseries[before_id]
    after_update = pres_timeseries[before_id+1]
    
    votes_djt_before = round(before_update["vote_shares"]["trumpd"] * before_update["votes"])
    votes_jrb_before = round(before_update["vote_shares"]["bidenj"] * before_update["votes"])
    votes_djt_after = round(after_update["vote_shares"]["trumpd"] * after_update["votes"])
    votes_jrb_after = round(after_update["vote_shares"]["bidenj"] * after_update["votes"])

    diff_djt = votes_djt_after - votes_djt_before
    diff_jrb = votes_jrb_after - votes_jrb_before
    diff_total = after_update["votes"] - before_update["votes"]
    
    
    print("DISCREPANCY DETECTED:\n")
    print("DJT VOTES BEFORE: {}".format(votes_djt_before))
    print("JRB VOTES BEFORE: {}".format(votes_jrb_before))
    print("TOTAL BEFORE: {}".format(before_update["votes"]))
    print("TIMESTAMP BEFORE: {}\n".format(before_update["timestamp"]))

    print("DJT VOTES AFTER: {}".format(votes_djt_after))
    print("JRB VOTES AFTER: {}".format(votes_jrb_after))
    print("TOTAL AFTER: {}".format(after_update["votes"]))
    print("TIMESTAMP AFTER: {}\n".format(after_update["timestamp"]))


    print("DJT DIFF: {}".format(diff_djt))
    print("JRB DIFF: {}".format(diff_jrb))
    print("TOTAL DIFF: {}".format(diff_total))
    print("===================================")
```


```python
for i,update in enumerate(pres_timeseries):
    before_update = pres_timeseries[i]
    after_update = pres_timeseries[i+1]
    
    # Calculate vote counts
    votes_djt_before = round(before_update["vote_shares"]["trumpd"] * before_update["votes"])
    votes_jrb_before = round(before_update["vote_shares"]["bidenj"] * before_update["votes"])
    votes_djt_after = round(after_update["vote_shares"]["trumpd"] * after_update["votes"])
    votes_jrb_after = round(after_update["vote_shares"]["bidenj"] * after_update["votes"])
    
    if((votes_djt_after < votes_djt_before) or (votes_jrb_after < votes_jrb_before)):
        print_discrepancy(i)
    
    if(i==len(pres_timeseries)-2):
        break
    
    
```

    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 33
    JRB VOTES BEFORE: 44
    TOTAL BEFORE: 77
    TIMESTAMP BEFORE: 2020-11-04T00:19:27Z
    
    DJT VOTES AFTER: 0
    JRB VOTES AFTER: 0
    TOTAL AFTER: 0
    TIMESTAMP AFTER: 2020-11-04T00:30:20Z
    
    DJT DIFF: -33
    JRB DIFF: -44
    TOTAL DIFF: -77
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 4
    JRB VOTES BEFORE: 7
    TOTAL BEFORE: 11
    TIMESTAMP BEFORE: 2020-11-04T00:42:45Z
    
    DJT VOTES AFTER: 0
    JRB VOTES AFTER: 0
    TOTAL AFTER: 0
    TIMESTAMP AFTER: 2020-11-04T00:59:15Z
    
    DJT DIFF: -4
    JRB DIFF: -7
    TOTAL DIFF: -11
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 37039
    JRB VOTES BEFORE: 180154
    TOTAL BEFORE: 219166
    TIMESTAMP BEFORE: 2020-11-04T01:17:19Z
    
    DJT VOTES AFTER: 37055
    JRB VOTES AFTER: 180011
    TOTAL AFTER: 219258
    TIMESTAMP AFTER: 2020-11-04T01:24:10Z
    
    DJT DIFF: 16
    JRB DIFF: -143
    TOTAL DIFF: 92
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 47567
    JRB VOTES BEFORE: 256553
    TOTAL BEFORE: 306881
    TIMESTAMP BEFORE: 2020-11-04T01:29:03Z
    
    DJT VOTES AFTER: 177822
    JRB VOTES AFTER: 256465
    TOTAL AFTER: 436908
    TIMESTAMP AFTER: 2020-11-04T01:31:11Z
    
    DJT DIFF: 130255
    JRB DIFF: -88
    TOTAL DIFF: 130027
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 390167
    JRB VOTES BEFORE: 712527
    TOTAL BEFORE: 1111586
    TIMESTAMP BEFORE: 2020-11-04T02:13:11Z
    
    DJT VOTES AFTER: 347841
    JRB VOTES AFTER: 516095
    TOTAL AFTER: 871782
    TIMESTAMP AFTER: 2020-11-04T02:14:32Z
    
    DJT DIFF: -42326
    JRB DIFF: -196432
    TOTAL DIFF: -239804
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 360896
    JRB VOTES BEFORE: 520900
    TOTAL BEFORE: 888907
    TIMESTAMP BEFORE: 2020-11-04T02:16:43Z
    
    DJT VOTES AFTER: 232980
    JRB VOTES AFTER: 533300
    TOTAL AFTER: 774021
    TIMESTAMP AFTER: 2020-11-04T02:17:03Z
    
    DJT DIFF: -127916
    JRB DIFF: 12400
    TOTAL DIFF: -114886
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 413597
    JRB VOTES BEFORE: 880220
    TOTAL BEFORE: 1325632
    TIMESTAMP BEFORE: 2020-11-04T02:21:59Z
    
    DJT VOTES AFTER: 268418
    JRB VOTES AFTER: 463631
    TOTAL AFTER: 739443
    TIMESTAMP AFTER: 2020-11-04T02:22:45Z
    
    DJT DIFF: -145179
    JRB DIFF: -416589
    TOTAL DIFF: -586189
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 425367
    JRB VOTES BEFORE: 556555
    TOTAL BEFORE: 993848
    TIMESTAMP BEFORE: 2020-11-04T02:38:07Z
    
    DJT VOTES AFTER: 425435
    JRB VOTES AFTER: 555649
    TOTAL AFTER: 994006
    TIMESTAMP AFTER: 2020-11-04T02:38:19Z
    
    DJT DIFF: 68
    JRB DIFF: -906
    TOTAL DIFF: 158
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 527082
    JRB VOTES BEFORE: 601399
    TOTAL BEFORE: 1143344
    TIMESTAMP BEFORE: 2020-11-04T02:47:35Z
    
    DJT VOTES AFTER: 528051
    JRB VOTES AFTER: 601359
    TOTAL AFTER: 1145446
    TIMESTAMP AFTER: 2020-11-04T02:47:57Z
    
    DJT DIFF: 969
    JRB DIFF: -40
    TOTAL DIFF: 2102
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 668911
    JRB VOTES BEFORE: 662168
    TOTAL BEFORE: 1348610
    TIMESTAMP BEFORE: 2020-11-04T02:55:22Z
    
    DJT VOTES AFTER: 671093
    JRB VOTES AFTER: 661641
    TOTAL AFTER: 1350287
    TIMESTAMP AFTER: 2020-11-04T02:55:34Z
    
    DJT DIFF: 2182
    JRB DIFF: -527
    TOTAL DIFF: 1677
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 937785
    JRB VOTES BEFORE: 805330
    TOTAL BEFORE: 1766074
    TIMESTAMP BEFORE: 2020-11-04T03:12:51Z
    
    DJT VOTES AFTER: 937898
    JRB VOTES AFTER: 803660
    TOTAL AFTER: 1766286
    TIMESTAMP AFTER: 2020-11-04T03:13:18Z
    
    DJT DIFF: 113
    JRB DIFF: -1670
    TOTAL DIFF: 212
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 1168854
    JRB VOTES BEFORE: 991556
    TOTAL BEFORE: 2188865
    TIMESTAMP BEFORE: 2020-11-04T03:29:30Z
    
    DJT VOTES AFTER: 1170935
    JRB VOTES AFTER: 991129
    TOTAL AFTER: 2192763
    TIMESTAMP AFTER: 2020-11-04T03:31:51Z
    
    DJT DIFF: 2081
    JRB DIFF: -427
    TOTAL DIFF: 3898
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 1219608
    JRB VOTES BEFORE: 1013700
    TOTAL BEFORE: 2262723
    TIMESTAMP BEFORE: 2020-11-04T03:33:51Z
    
    DJT VOTES AFTER: 1221405
    JRB VOTES AFTER: 1012928
    TOTAL AFTER: 2266058
    TIMESTAMP AFTER: 2020-11-04T03:34:13Z
    
    DJT DIFF: 1797
    JRB DIFF: -772
    TOTAL DIFF: 3335
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 1232133
    JRB VOTES BEFORE: 1024111
    TOTAL BEFORE: 2285961
    TIMESTAMP BEFORE: 2020-11-04T03:36:12Z
    
    DJT VOTES AFTER: 1229992
    JRB VOTES AFTER: 1024231
    TOTAL AFTER: 2286231
    TIMESTAMP AFTER: 2020-11-04T03:37:38Z
    
    DJT DIFF: -2141
    JRB DIFF: 120
    TOTAL DIFF: 270
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 1278284
    JRB VOTES BEFORE: 1042873
    TOTAL BEFORE: 2354114
    TIMESTAMP BEFORE: 2020-11-04T03:40:27Z
    
    DJT VOTES AFTER: 1283097
    JRB VOTES AFTER: 1042516
    TOTAL AFTER: 2358634
    TIMESTAMP AFTER: 2020-11-04T03:42:23Z
    
    DJT DIFF: 4813
    JRB DIFF: -357
    TOTAL DIFF: 4520
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 1655410
    JRB VOTES BEFORE: 1212413
    TOTAL BEFORE: 2914454
    TIMESTAMP BEFORE: 2020-11-04T03:59:27Z
    
    DJT VOTES AFTER: 1656175
    JRB VOTES AFTER: 1210057
    TOTAL AFTER: 2915801
    TIMESTAMP AFTER: 2020-11-04T04:00:36Z
    
    DJT DIFF: 765
    JRB DIFF: -2356
    TOTAL DIFF: 1347
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 1689209
    JRB VOTES BEFORE: 1253477
    TOTAL BEFORE: 2984468
    TIMESTAMP BEFORE: 2020-11-04T04:07:43Z
    
    DJT VOTES AFTER: 1671332
    JRB VOTES AFTER: 1271406
    TOTAL AFTER: 2984522
    TIMESTAMP AFTER: 2020-11-04T04:08:51Z
    
    DJT DIFF: -17877
    JRB DIFF: 17929
    TOTAL DIFF: 54
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 1688506
    JRB VOTES BEFORE: 1279171
    TOTAL BEFORE: 3009814
    TIMESTAMP BEFORE: 2020-11-04T04:11:10Z
    
    DJT VOTES AFTER: 1693503
    JRB VOTES AFTER: 1277661
    TOTAL AFTER: 3013351
    TIMESTAMP AFTER: 2020-11-04T04:12:11Z
    
    DJT DIFF: 4997
    JRB DIFF: -1510
    TOTAL DIFF: 3537
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 2019991
    JRB VOTES BEFORE: 1511412
    TOTAL BEFORE: 3581545
    TIMESTAMP BEFORE: 2020-11-04T04:41:32Z
    
    DJT VOTES AFTER: 2021054
    JRB VOTES AFTER: 1508624
    TOTAL AFTER: 3583429
    TIMESTAMP AFTER: 2020-11-04T04:41:57Z
    
    DJT DIFF: 1063
    JRB DIFF: -2788
    TOTAL DIFF: 1884
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 2592227
    JRB VOTES BEFORE: 1912224
    TOTAL BEFORE: 4563780
    TIMESTAMP BEFORE: 2020-11-04T05:52:34Z
    
    DJT VOTES AFTER: 2591374
    JRB VOTES AFTER: 1922928
    TOTAL AFTER: 4578399
    TIMESTAMP AFTER: 2020-11-04T05:53:08Z
    
    DJT DIFF: -853
    JRB DIFF: 10704
    TOTAL DIFF: 14619
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 2855387
    JRB VOTES BEFORE: 2159323
    TOTAL BEFORE: 5080760
    TIMESTAMP BEFORE: 2020-11-04T07:06:51Z
    
    DJT VOTES AFTER: 2857822
    JRB VOTES AFTER: 2156079
    TOTAL AFTER: 5085092
    TIMESTAMP AFTER: 2020-11-04T07:08:49Z
    
    DJT DIFF: 2435
    JRB DIFF: -3244
    TOTAL DIFF: 4332
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 2971368
    JRB VOTES BEFORE: 2351209
    TOTAL BEFORE: 5392682
    TIMESTAMP BEFORE: 2020-11-04T10:54:40Z
    
    DJT VOTES AFTER: 2968494
    JRB VOTES AFTER: 2353206
    TOTAL AFTER: 5397262
    TIMESTAMP AFTER: 2020-11-04T14:16:51Z
    
    DJT DIFF: -2874
    JRB DIFF: 1997
    TOTAL DIFF: 4580
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 2979427
    JRB VOTES BEFORE: 2386804
    TOTAL BEFORE: 5436911
    TIMESTAMP BEFORE: 2020-11-04T14:54:50Z
    
    DJT VOTES AFTER: 2976329
    JRB VOTES AFTER: 2388681
    TOTAL AFTER: 5441186
    TIMESTAMP AFTER: 2020-11-04T15:00:52Z
    
    DJT DIFF: -3098
    JRB DIFF: 1877
    TOTAL DIFF: 4275
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3067023
    JRB VOTES BEFORE: 2601801
    TOTAL BEFORE: 5743489
    TIMESTAMP BEFORE: 2020-11-04T19:04:59Z
    
    DJT VOTES AFTER: 3066905
    JRB VOTES AFTER: 2612336
    TOTAL AFTER: 5754044
    TIMESTAMP AFTER: 2020-11-04T19:16:54Z
    
    DJT DIFF: -118
    JRB DIFF: 10535
    TOTAL DIFF: 10555
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3074501
    JRB VOTES BEFORE: 2618806
    TOTAL BEFORE: 5768295
    TIMESTAMP BEFORE: 2020-11-04T19:42:52Z
    
    DJT VOTES AFTER: 3072281
    JRB VOTES AFTER: 2627609
    TOTAL AFTER: 5774964
    TIMESTAMP AFTER: 2020-11-04T19:45:52Z
    
    DJT DIFF: -2220
    JRB DIFF: 8803
    TOTAL DIFF: 6669
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3099697
    JRB VOTES BEFORE: 2738854
    TOTAL BEFORE: 5915452
    TIMESTAMP BEFORE: 2020-11-04T20:45:58Z
    
    DJT VOTES AFTER: 3096601
    JRB VOTES AFTER: 2747272
    TOTAL AFTER: 5920844
    TIMESTAMP AFTER: 2020-11-04T20:49:00Z
    
    DJT DIFF: -3096
    JRB DIFF: 8418
    TOTAL DIFF: 5392
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3124031
    JRB VOTES BEFORE: 2811627
    TOTAL BEFORE: 6007751
    TIMESTAMP BEFORE: 2020-11-04T22:01:43Z
    
    DJT VOTES AFTER: 3122774
    JRB VOTES AFTER: 2815912
    TOTAL AFTER: 6016906
    TIMESTAMP AFTER: 2020-11-04T22:03:40Z
    
    DJT DIFF: -1257
    JRB DIFF: 4285
    TOTAL DIFF: 9155
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3128836
    JRB VOTES BEFORE: 2827406
    TOTAL BEFORE: 6028585
    TIMESTAMP BEFORE: 2020-11-04T22:18:53Z
    
    DJT VOTES AFTER: 3130032
    JRB VOTES AFTER: 2822457
    TOTAL AFTER: 6030891
    TIMESTAMP AFTER: 2020-11-04T22:19:53Z
    
    DJT DIFF: 1196
    JRB DIFF: -4949
    TOTAL DIFF: 2306
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3130032
    JRB VOTES BEFORE: 2822457
    TOTAL BEFORE: 6030891
    TIMESTAMP BEFORE: 2020-11-04T22:19:53Z
    
    DJT VOTES AFTER: 3129455
    JRB VOTES AFTER: 2833425
    TOTAL AFTER: 6041418
    TIMESTAMP AFTER: 2020-11-04T22:36:54Z
    
    DJT DIFF: -577
    JRB DIFF: 10968
    TOTAL DIFF: 10527
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3136704
    JRB VOTES BEFORE: 2846044
    TOTAL BEFORE: 6055413
    TIMESTAMP BEFORE: 2020-11-04T22:41:51Z
    
    DJT VOTES AFTER: 3137930
    JRB VOTES AFTER: 2841098
    TOTAL AFTER: 6057779
    TIMESTAMP AFTER: 2020-11-04T22:42:49Z
    
    DJT DIFF: 1226
    JRB DIFF: -4946
    TOTAL DIFF: 2366
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3138637
    JRB VOTES BEFORE: 2841739
    TOTAL BEFORE: 6059144
    TIMESTAMP BEFORE: 2020-11-04T22:52:52Z
    
    DJT VOTES AFTER: 3137411
    JRB VOTES AFTER: 2852192
    TOTAL AFTER: 6068493
    TIMESTAMP AFTER: 2020-11-04T22:54:49Z
    
    DJT DIFF: -1226
    JRB DIFF: 10453
    TOTAL DIFF: 9349
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3149879
    JRB VOTES BEFORE: 2881284
    TOTAL BEFORE: 6104416
    TIMESTAMP BEFORE: 2020-11-04T23:12:52Z
    
    DJT VOTES AFTER: 3146132
    JRB VOTES AFTER: 2883446
    TOTAL AFTER: 6108995
    TIMESTAMP AFTER: 2020-11-04T23:14:52Z
    
    DJT DIFF: -3747
    JRB DIFF: 2162
    TOTAL DIFF: 4579
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3176641
    JRB VOTES BEFORE: 2941334
    TOTAL BEFORE: 6192283
    TIMESTAMP BEFORE: 2020-11-05T00:28:52Z
    
    DJT VOTES AFTER: 3175778
    JRB VOTES AFTER: 2946279
    TOTAL AFTER: 6202692
    TIMESTAMP AFTER: 2020-11-05T00:34:29Z
    
    DJT DIFF: -863
    JRB DIFF: 4945
    TOTAL DIFF: 10409
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3186701
    JRB VOTES BEFORE: 2974670
    TOTAL BEFORE: 6236205
    TIMESTAMP BEFORE: 2020-11-05T00:39:58Z
    
    DJT VOTES AFTER: 3184493
    JRB VOTES AFTER: 2978438
    TOTAL AFTER: 6244104
    TIMESTAMP AFTER: 2020-11-05T00:46:50Z
    
    DJT DIFF: -2208
    JRB DIFF: 3768
    TOTAL DIFF: 7899
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3195938
    JRB VOTES BEFORE: 2995409
    TOTAL BEFORE: 6266545
    TIMESTAMP BEFORE: 2020-11-05T01:03:50Z
    
    DJT VOTES AFTER: 3195062
    JRB VOTES AFTER: 3000471
    TOTAL AFTER: 6277135
    TIMESTAMP AFTER: 2020-11-05T01:20:52Z
    
    DJT DIFF: -876
    JRB DIFF: 5062
    TOTAL DIFF: 10590
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3200731
    JRB VOTES BEFORE: 3005794
    TOTAL BEFORE: 6288273
    TIMESTAMP BEFORE: 2020-11-05T02:18:57Z
    
    DJT VOTES AFTER: 3199611
    JRB VOTES AFTER: 3016956
    TOTAL AFTER: 6298446
    TIMESTAMP AFTER: 2020-11-05T02:44:56Z
    
    DJT DIFF: -1120
    JRB DIFF: 11162
    TOTAL DIFF: 10173
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3220020
    JRB VOTES BEFORE: 3054891
    TOTAL BEFORE: 6351124
    TIMESTAMP BEFORE: 2020-11-05T13:58:46Z
    
    DJT VOTES AFTER: 3217763
    JRB VOTES AFTER: 3071211
    TOTAL AFTER: 6371807
    TIMESTAMP AFTER: 2020-11-05T14:04:26Z
    
    DJT DIFF: -2257
    JRB DIFF: 16320
    TOTAL DIFF: 20683
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3233161
    JRB VOTES BEFORE: 3117461
    TOTAL BEFORE: 6427755
    TIMESTAMP BEFORE: 2020-11-05T18:12:45Z
    
    DJT VOTES AFTER: 3227963
    JRB VOTES AFTER: 3118650
    TOTAL AFTER: 6430206
    TIMESTAMP AFTER: 2020-11-05T18:18:46Z
    
    DJT DIFF: -5198
    JRB DIFF: 1189
    TOTAL DIFF: 2451
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3238713
    JRB VOTES BEFORE: 3129036
    TOTAL BEFORE: 6451620
    TIMESTAMP BEFORE: 2020-11-05T20:44:39Z
    
    DJT VOTES AFTER: 3235860
    JRB VOTES AFTER: 3138978
    TOTAL AFTER: 6458803
    TIMESTAMP AFTER: 2020-11-05T21:03:39Z
    
    DJT DIFF: -2853
    JRB DIFF: 9942
    TOTAL DIFF: 7183
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3262742
    JRB VOTES BEFORE: 3184436
    TOTAL BEFORE: 6525483
    TIMESTAMP BEFORE: 2020-11-05T23:14:13Z
    
    DJT VOTES AFTER: 3258420
    JRB VOTES AFTER: 3186591
    TOTAL AFTER: 6529899
    TIMESTAMP AFTER: 2020-11-05T23:26:53Z
    
    DJT DIFF: -4322
    JRB DIFF: 2155
    TOTAL DIFF: 4416
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3267121
    JRB VOTES BEFORE: 3201648
    TOTAL BEFORE: 6547337
    TIMESTAMP BEFORE: 2020-11-06T00:12:10Z
    
    DJT VOTES AFTER: 3260786
    JRB VOTES AFTER: 3201856
    TOTAL AFTER: 6547763
    TIMESTAMP AFTER: 2020-11-06T00:20:12Z
    
    DJT DIFF: -6335
    JRB DIFF: 208
    TOTAL DIFF: 426
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3272692
    JRB VOTES BEFORE: 3220119
    TOTAL BEFORE: 6571671
    TIMESTAMP BEFORE: 2020-11-06T01:32:40Z
    
    DJT VOTES AFTER: 3269835
    JRB VOTES AFTER: 3223781
    TOTAL AFTER: 6579145
    TIMESTAMP AFTER: 2020-11-06T01:50:36Z
    
    DJT DIFF: -2857
    JRB DIFF: 3662
    TOTAL DIFF: 7474
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3280578
    JRB VOTES BEFORE: 3234373
    TOTAL BEFORE: 6600761
    TIMESTAMP BEFORE: 2020-11-06T03:12:40Z
    
    DJT VOTES AFTER: 3277968
    JRB VOTES AFTER: 3244924
    TOTAL AFTER: 6608807
    TIMESTAMP AFTER: 2020-11-06T03:32:40Z
    
    DJT DIFF: -2610
    JRB DIFF: 10551
    TOTAL DIFF: 8046
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3284867
    JRB VOTES BEFORE: 3258376
    TOTAL BEFORE: 6622716
    TIMESTAMP BEFORE: 2020-11-06T03:42:34Z
    
    DJT VOTES AFTER: 3280158
    JRB VOTES AFTER: 3260278
    TOTAL AFTER: 6626581
    TIMESTAMP AFTER: 2020-11-06T04:28:44Z
    
    DJT DIFF: -4709
    JRB DIFF: 1902
    TOTAL DIFF: 3865
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3286435
    JRB VOTES BEFORE: 3266517
    TOTAL BEFORE: 6639262
    TIMESTAMP BEFORE: 2020-11-06T06:30:10Z
    
    DJT VOTES AFTER: 3286296
    JRB VOTES AFTER: 3266379
    TOTAL AFTER: 6638982
    TIMESTAMP AFTER: 2020-11-06T12:22:44Z
    
    DJT DIFF: -139
    JRB DIFF: -138
    TOTAL DIFF: -280
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3305179
    JRB VOTES BEFORE: 3318587
    TOTAL BEFORE: 6704217
    TIMESTAMP BEFORE: 2020-11-06T23:24:40Z
    
    DJT VOTES AFTER: 3298742
    JRB VOTES AFTER: 3318856
    TOTAL AFTER: 6704760
    TIMESTAMP AFTER: 2020-11-06T23:38:39Z
    
    DJT DIFF: -6437
    JRB DIFF: 269
    TOTAL DIFF: 543
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3310508
    JRB VOTES BEFORE: 3337422
    TOTAL BEFORE: 6728674
    TIMESTAMP BEFORE: 2020-11-07T02:40:18Z
    
    DJT VOTES AFTER: 3305351
    JRB VOTES AFTER: 3339010
    TOTAL AFTER: 6731875
    TIMESTAMP AFTER: 2020-11-07T03:40:45Z
    
    DJT DIFF: -5157
    JRB DIFF: 1588
    TOTAL DIFF: 3201
    ===================================
    DISCREPANCY DETECTED:
    
    DJT VOTES BEFORE: 3317639
    JRB VOTES BEFORE: 3358181
    TOTAL BEFORE: 6756903
    TIMESTAMP BEFORE: 2020-11-08T18:36:39Z
    
    DJT VOTES AFTER: 3311557
    JRB VOTES AFTER: 3358865
    TOTAL AFTER: 6758279
    TIMESTAMP AFTER: 2020-11-08T19:58:46Z
    
    DJT DIFF: -6082
    JRB DIFF: 684
    TOTAL DIFF: 1376
    ===================================



```python

```
