---
layout: post
title:  "Imagick: Automatic line wrapping"
date:   2018-02-28 07:48:50 +0000
categories: php
tags: php imagick
blurb: Imagick doesn't have the ability to automatically wrap lines when outputting text, which is a problem when trying to render large amounts of content.
---

Imagick doesn't have the ability to automatically wrap lines when outputting text, which was an issue when I was trying to render text when given several paragraphs.  The requirement was to be able to wrap these lines whilst also honouring any newlines that had been added as well.

To start off with, we need to split up our content into an array with each word as a separate value.

```php
$imagick = new Imagick();
$imagickDraw = new ImagickDraw();

$boxWidth = 500;
$boxHeight = 500;
$boxX = 0;
$boxY = 0;
$lineHeight = 0;

$content = trim($content);
$words = explode(' ', $content);
```

We can then loop through these words and begin to process them.  By the end, we should end up with an array containing a line per array value.

The first thing to do is to sort out any line breaks that already exist.  If there are any line breaks, these will be stored as PHP_EOL characters inside the array values.  We also need to ensure that we keep multiple line breaks if they exist.  Below is the code sample to store the words an array of lines.

```php
$lines = [];
$currentLine = [];
foreach ($words as $word) {
    $newLines = [];

    if (substr_count($word, PHP_EOL) > 1) {
        // The word contains multiple line breaks, so we need to ensure they are adhered to
        $newLines = array_fill(0, substr_count($word, PHP_EOL) - 1, '');
    }

    if (strpos($word, PHP_EOL) === 0) {
        // Newline characters are at the start of the word
        $lines[] = implode(' ', $line);
        $lines = array_merge($lines, $newLines);
        $currentLine = [trim($word)];
    } elseif (strpos($word, PHP_EOL) > 0) {
        // Newline characters are at the end of the word
        $line[] = trim($word);
        $lines[] = implode(' ', $line);
        $lines = array_merge($lines, $newLines);
        $line = [];
    } else {
        $line[] = trim($word);

        $metrics = $imagick->queryFontMetrics($imagickDraw, implode(' ', $line));
        $lineHeight = max($metrics['textHeight'], $lineHeight);

        if ($metrics['textWidth'] > $boxWidth)
        {
            if (count($line) == 1) {
                // There is only one word on this line and it is too wide, but we will just add it anyway
                $lines[] = $line[0];
                $line = [];
            } else {
                // The last word added made the line too wide, so we remove the word and store the line as it was
                array_pop($line);
                $lines[] = implode(' ', $line);

                $line = [$word];
            }
        }

        // If there are no more words to process, add the last line to the array
        if (next($words) === false) {
            $lines[] = implode(' ', $line);
        }
    }
}
```

We use the `queryFontMetrics` method to determine whether we can fit any more words onto the line. This method returns what the current width in pixels is of the line so far.  We can compare that with our desired box width to see whether we need to finish that line and start a new one.

We can now use this line array to draw these lines onto the imagick canvas.

```php
for($i = 0; $i < count($lines); $i++) {
    $topOfLinePosition = $i * $lineHeight;

    if ($topOfLinePosition < $boxHeight) {
        $boxY = $boxY + $topOfLinePosition;
        $imagick->annotateImage($imagickDraw, $boxX, $boxY + $imagickDraw->getFontSize(), 0, $lines[$i]);
    }
}
```

We use the line height calculated use the font metrics to determine where to draw the first line.  After that we add the height of the font to this line height to ensure that each line does not overlap.

The code is obviously simplified and can be extracted into separate functions but it should work as-is.
