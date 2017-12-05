#pipeline for finding lane lines on the road
1. do make gray scale from source image
2. do gaussian blur for scattered point
3. do determine edges by canny detect algrithm
4. to be given interested region by polygon (four vertices)
5. do determine straight lines by hough algorithm, then get rid of short lines
6. do draw long continue lines 

#shortcomings
1. for normal test videos and challenge video, i use different parameters, for examples, canny threshold, hough min line length, interested region in function 'region_of_interest', but i use parameter with global variables, i think it not so good

2. event if i use different parameters, but output of challenge vedio is very poor effect, i think i use solid region is bad method in function 'region_of_interest', in challenge vedio, region of lane lines is not fixed and is changed rapidly. i don't known how to do this

3. in function 'draw_lines':

```
	for line in lines:	
        for x1,y1,x2,y2 in line:
            cv2.line(img, (x1, y1), (x2, y2), color, thickness)
            #for interplote line;
            if abs(y2 - y1) > 30.0 and abs(x2-x1) > 0.000001:
                k = (y2-y1)/(x2-x1)
                b = y2 - k*x2
                py = img.shape[0]
                px = int((py - b)/k)
                if y1 < y2:
                    cv2.line(img, (x1, y1), (px, py), color, thickness)
                else:
                    cv2.line(img, (x2, y2), (px, py), color, thickness)
```

in above code, i interpolate lines with result of hough processing. but use magic code 30 and 0.0000001, i think some line too short to be continue and some line's slope rate is zero. by two points of line, i calculate slope rate and intercept, then i could get intersection point with lane line and section bottom
i think this is not so good method?

#questions
1. trying challenge video, i found some problems. i scratch some images from challenge for verifying pipeline(for testing challenge images: search 'for challenge images' in P1.ipynb file, output is in test_challenge_images directory). code for scratch images of chanllege video:

```
count = 0
def process_image(image):
    global count
    # NOTE: The output you return should be a color image (3 channel) for processing video below
    # TODO: put your pipeline here,
    # you should return the final output (image where lines are drawn on lanes)
    result,_ = find_lane_lines(image)
     count = count + 1
     if count % 50 == 0:
         mpimg.imsave('test_challenge_images/%05d.jpg' % count, image)
    return result
```

when i use above scratched images from challenge video, pipeline's code don't run normally and report error: not same channel in 'hough_lines' function: 

```
line_img = np.zeros((img.shape[0], img.shape[1], 3), dtype=np.uint8)
```
then i found scratched images from challenge video is four channel, but hough_lines function used solid channel three, so i changed related code and add parameter 'source_image' for getting channel number of source image:

```
	def hough_lines(source_image, img, rho, theta, threshold, min_line_len, max_line_gap):
	...
	channel = 3
    shape = source_image.shape
    if len(shape) > 2:
        channel = shape[2]
    line_img = np.zeros((img.shape[0], img.shape[1], channel), dtype=np.uint8)
    draw_lines(line_img, lines)
    return line_img
```
am i correct? i don't know.

2. my output of challenge video is poor effect(result in test_videos), i have no more better method to improve, techer can give some advances?

#improvments

1. result of the 'hough_lines' function have multiple lines, currently, i filtered those lines by line's slope, left and right only selected one line as lane lines. if i mixed mutiple nearby lines into one line and the effect should be more better.