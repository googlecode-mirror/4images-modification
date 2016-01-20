<?php
/
  * *** 4images - A Web Based Image Gallery Management System**
  * ----------------------------------------------------------------    *****
  * File: admin\_functions.php                                  *** Copyright: (C) 2002-2012 Jan Sorgalla**
  * Email: jan@4homepages.de                                    *** Web: http://www.4homepages.de**
  * Scriptversion: 1.7.11                                               *****
  * Never released without support from: Nicky (http://www.nicky.net)   *****
  * 
  * *** Dieses Script ist KEINE Freeware. Bitte lesen Sie die Lizenz-**
  * bedingungen (Lizenz.txt) für weitere Informationen.                 *** ---------------------------------------------------------------**
  * This script is NOT freeware! Please read the Copyright Notice       *** (Licence.txt) for further information.**
  * *****/

function get\_iptc\_insert\_link($file, $iptc\_tag, $input, $add\_text = 1) {
> global $lang;
> if ($imageinfo = @getimagesize($file, $info)) {
> > if (isset($info['APP13'])) {
> > > $iptc = iptcparse($info['APP13']);


> if (is\_array($iptc)) {
> > $separator = "";
> > switch ($iptc\_tag) {
> > case "caption":
> > > if (isset($iptc['2#120'][0](0.md))) {
> > > > $value = trim($iptc['2#120'][0](0.md));

> > > }
> > > break;


> case "date\_created":
> > if (isset($iptc['2#055'][0](0.md))) {
> > > $value = $iptc['2#055'][0](0.md);
> > > $value = preg\_replace("/([0-9]{4})([0-9]{2})([0-9]{2})/", "\\1-\\2-\\3", $value);

> > }
> > break;


> case "keyword":
> > $value = "";
> > if (isset($iptc['2#025'])) {
> > > foreach ($iptc['2#025'] as $val) {
> > > > $value .= (($value != "" ) ? "," : "").trim($val);

> > > }

> > }
> > $separator = ",";
> > break;


> case "object\_name":
> > if (isset($iptc['2#005'][0](0.md))) {
> > > $value = trim($iptc['2#005'][0](0.md));

> > }
> > break;


> default:
> > $value = "";
> > break;

> }
> if (!empty($value)) {
> > $html = "\n<input type=\"hidden\" name=\"hidden_".$input."\" value=\"".$value." \">\n";
> > $html .= "_

&lt;script language=\"javascript\"&gt;

\n<!--\n";
> > $html .= "this.document.writeln('<br /><br /><input type=\"button\" value=\"IPTC ".str\_replace(":", "", $lang['iptc_'.$iptc\_tag])." &raquo;\" onClick=\"this.form.".$input.".value=".(($add\_text) ? "this.form.".$input.".value + ".($separator ? "(this.form.".$input.".value.replace(/^\s+/, \'\').replace(/\s+$/, \'\') == \'\' ? \'\' : \'".$separator."\')+" : "") : "")."this.form.hidden_".$input.".value\">');";
> > $html .= "\n//-->\n

&lt;/script&gt;

\n";
> > return $html;

> }
> }
> }
> }
}

function copy\_media($image\_media\_file, $from\_cat = 0, $to\_cat = 0) {
> global $config;

> if (is\_remote($image\_media\_file)) {
> > return $image\_media\_file;

> }

> $image\_src = ($from\_cat != -1) ? MEDIA\_PATH.(($from\_cat != 0) ? "/".$from\_cat : "") : MEDIA\_TEMP\_PATH;
> $image\_dest = ($to\_cat != -1) ? MEDIA\_PATH.(($to\_cat != 0) ? "/".$to\_cat : "") : MEDIA\_TEMP\_PATH;
> return copy\_file($image\_src, $image\_dest, $image\_media\_file, $image\_media\_file, $config['upload\_mode']);
}


function copy\_thumbnail($image\_media\_file, $image\_thumb\_file, $from\_cat = 0, $to\_cat = 0) {
> if (is\_remote($image\_thumb\_file)) {
> > return $image\_thumb\_file;

> }

> $thumb\_src = ($from\_cat != -1) ? THUMB\_PATH.(($from\_cat != 0) ? "/".$from\_cat : "") : THUMB\_TEMP\_PATH;
> $thumb\_dest = ($to\_cat != -1) ? THUMB\_PATH.(($to\_cat != 0) ? "/".$to\_cat : "") : THUMB\_TEMP\_PATH;

> if ($image\_thumb\_file != "" && file\_exists($thumb\_src."/".$image\_thumb\_file)) {
> > $thumb\_extension = get\_file\_extension($image\_thumb\_file);
> > $new\_thumb = get\_file\_name($image\_media\_file).".".$thumb\_extension;
> > if ($new\_thumb = copy\_file($thumb\_src, $thumb\_dest, $image\_thumb\_file, $new\_thumb, 1))
> > {
> > > $image\_thumb\_file = $new\_thumb;

> > }

> }
> return $image\_thumb\_file;
}

function copy\_file($image\_src, $image\_dest, $image\_media\_file, $dest\_file\_name, $type, $filter = 1, $move = 1)
{
> $image\_src\_file = $image\_src."/".$image\_media\_file;
> $dest\_file\_name = ($filter) ? filterFileName($dest\_file\_name) : $dest\_file\_name;
> $ok = 0;
> if (!file\_exists($image\_dest) || !is\_dir($image\_dest))
> {
> $oldumask = umask(0);
> $result = _mkdir($image\_dest);
> > @chmod($image\_dest, CHMOD\_DIRS);

> umask($oldumask);
> }
> switch ($type) {
> case 1: // overwrite mode
> > if (file\_exists($image\_src."/".$image\_media\_file)) {
> > > if (file\_exists($image\_dest."/".$dest\_file\_name)) {
> > > > unlink($image\_dest."/".$dest\_file\_name);

> > > }
> > > $ok = copy($image\_src."/".$image\_media\_file, $image\_dest."/".$dest\_file\_name);

> > }
> > break;_


> case 2: // create new with incremental extention
> > if (file\_exists($image\_src."/".$image\_media\_file)) {
> > > $file\_extension = get\_file\_extension($dest\_file\_name);
> > > $file\_name = get\_file\_name($dest\_file\_name);


> $n = 2;
> $copy = "";
> while (file\_exists($image\_dest."/".$file\_name.$copy.".".$file\_extension)) {
> > $copy = "_".$n;
> > $n++;

> }
> $new\_file = $file\_name.$copy.".".$file\_extension;
> $ok = copy($image\_src."/".$image\_media\_file, $image\_dest."/".$new\_file);
> $dest\_file\_name = $new\_file;
> }
> break;_

> case 3: // do nothing if exists, highest protection
> default:
> > if (file\_exists($image\_src."/".$image\_media\_file)) {
> > > if (file\_exists($image\_dest."/".$dest\_file\_name)) {
> > > > $ok = 0;

> > > }
> > > else {
> > > > $ok = copy($image\_src."/".$image\_media\_file, $image\_dest."/".$dest\_file\_name);

> > > }

> > }
> > break;

> }

> if ($ok) {
> > if ($move)
> > {
> > > @unlink($image\_src\_file);

> > }
> > @chmod($image\_dest."/".$dest\_file\_name, CHMOD\_FILES);
> > return $dest\_file\_name;

> }
> else {
> > return false;

> }
}

function show\_admin\_header($headinsert = "") {
> global $newlangfile, $config, $old\_language\_dir, $self\_url, $site\_sess, $lang;

> header ("Cache-Control: no-store, no-cache, must-revalidate");
> header ("Cache-Control: pre-check=0, post-check=0, max-age=0", false);
> header ("Pragma: no-cache");
> header ("Expires: " . gmdate("D, d M Y H:i:s", time()) . " GMT");
> header ("Last-Modified: " . gmdate("D, d M Y H:i:s") . " GMT");

> $onload = "";
> if ($newlangfile && strstr($self\_url, "settings.php") === false) {
> > $browser\_language = getenv('HTTP\_ACCEPT\_LANGUAGE');
> > if (strstr($browser\_language, "de") !== false) {
> > > $alert\_msg = "Ihr in der Konfiguration angegebenes Language-Pack \\'\\'$old\_language\_dir\\'\\' wurde nicht gefunden. Bitte ändern Sie Ihre Spracheinstellungen unter \\'\\'Konfiguration -> Einstellungen -> Allgemeine Einstellungen\\'\\'.\\n Es wurde folgendes Pack gefunden und verwendet: \\'\\'$config[language\_dir](language_dir.md)\\'\\'.";

> > }
> > elseif (preg\_match("/s(b|r)/", $browser\_language)) {
> > > $alert\_msg = "Vas Language-Pack \\'\\'$old\_language\_dir\\'\\' u konfiguraciji nemoze da bude nadjen. Molimo Vas da promenite ispod \\'\\'Konfiguracija -> Promena -> Generalne Promene\\'\\'.\\n Trenutno je sledeci jezik pronadjen i bice koriscen: \\'\\'$config[language\_dir](language_dir.md)\\'\\'.\\n\\nTranslation sponsored by: Nicky (http://www.nicky.net)";

> > }
> > else {
> > > $alert\_msg = "Your configured language-pack \\'\\'$old\_language\_dir\\'\\' was not found. Please modify your language settings under \\'\\'Configuration -> Settings -> General Settings\\'\\'. The following language-pack was found and used: \\'\\'$config[language\_dir](language_dir.md)\\'\\'";

> > }
> > $onload = " onload=\"javascript:alert('$alert\_msg')\"";

> }
?>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">


&lt;html dir="&lt;?php echo $lang['direction']; ?&gt;"&gt;


> 

&lt;head&gt;


> > 

&lt;title&gt;

4images - Control Panel

&lt;/title&gt;


> > 

&lt;meta http-equiv="Content-Type" content="text/html; charset=&lt;?php echo $lang['charset']; ?&gt;"&gt;


> > 

&lt;link rel="stylesheet" href="&lt;?php echo ROOT\_PATH; ?&gt;admin/cpstyle.css"&gt;


> > <?php
> > echo $headinsert;
> > ?>
> > 

&lt;script language="JavaScript"&gt;


> > <!--
> > var statusWin, toppos, leftpos;
> > toppos = (screen.height - 401)/2;
> > leftpos = (screen.width - 401)/2;
> > function showProgress() {
> > > statusWin = window.open('<?php echo $site\_sess->url("progress.php"); ?>','Status','height=150,width=350,top='+toppos+',left='+leftpos+',location=no,scrollbars=no,menubars=no,toolbars=no,resizable=yes');
> > > statusWin.focus();

> > }


> function hideProgress() {
> > if (statusWin != null) {
> > > if (!statusWin.closed) {
> > > > statusWin.close();

> > > }

> > }

> }
> function CheckAll() {
> > for (var i=0;i<document.form.elements.length;i++) {
> > > var e = document.form.elements[i](i.md);
> > > if ((e.name != 'allbox') && (e.type=='checkbox')) {
> > > > e.checked = document.form.allbox.checked;

> > > }

> > }

> }

> function CheckCheckAll() {
> > var TotalBoxes = 0;
> > var TotalOn = 0;
> > for (var i=0;i<document.form.elements.length;i++) {
> > > var e = document.form.elements[i](i.md);
> > > if ((e.name != 'allbox') && (e.type=='checkbox')) {
> > > > TotalBoxes++;
> > > > if (e.checked) {
> > > > > TotalOn++;

> > > > }

> > > }

> > }
> > if (TotalBoxes==TotalOn) {
> > > document.form.allbox.checked=true;

> > }
> > else {
> > > document.form.allbox.checked=false;

> > }

> }
> // -->
> 

&lt;/script&gt;


> 

&lt;script type="text/javascript" language="javascript" src="&lt;?php echo ROOT\_PATH; ?&gt;admin/browserSniffer.js"&gt;



&lt;/script&gt;


> 

&lt;script type="text/javascript" language="javascript" src="&lt;?php echo ROOT\_PATH; ?&gt;admin/calendar.js"&gt;



&lt;/script&gt;


> 

Unknown end tag for &lt;/head&gt;


> <body leftmargin="20" topmargin="20" marginwidth="20" marginheight="20" bgcolor="#FFFFFF" text="#0F5475" link="#0F5475" vlink="#0F5475" alink="#0F5475"<?php echo $onload; ?>>
<?php
}

function show\_admin\_footer() {
> global $site\_db, $start\_time, $alluserinfo, $do\_gzip\_compress, $nozip, $config;
?>
> 

Unknown end tag for &lt;/body&gt;




Unknown end tag for &lt;/html&gt;


<?php
> $site\_db->close();
> if ($do\_gzip\_compress) {
> > if (preg\_match("/gzip/i", $HTTP\_SERVER\_VARS["HTTP\_ACCEPT\_ENCODING"])) {
> > > $encoding = "gzip";

> > }
> > elseif (preg\_match("/x-gzip/i", $HTTP\_SERVER\_VARS["HTTP\_ACCEPT\_ENCODING"])) {
> > > $encoding = "x-gzip";

> > }


> $gzip\_contents = ob\_get\_contents();
> ob\_end\_clean();

> if (defined("PRINT\_STATS") && PRINT\_STATS == 1){
> > $s = sprintf ("<!-- Use Encoding:         %s -->\n", $encoding);
> > $s .= sprintf("<!-- Not compress length:  %s -->\n", strlen($gzip\_contents));
> > $s .= sprintf("<!-- Compressed length:    %s -->\n", strlen(gzcompress($gzip\_contents, $config['gz\_compress\_level'])));
> > $gzip\_contents .= $s;

> }

> $gzip\_size = strlen($gzip\_contents);
> $gzip\_crc = crc32($gzip\_contents);

> $gzip\_contents = gzcompress($gzip\_contents, $config['gz\_compress\_level']);
> $gzip\_contents = substr($gzip\_contents, 0, strlen($gzip\_contents) - 4);

> header("Content-Encoding: $encoding");
> echo "\x1f\x8b\x08\x00\x00\x00\x00\x00";
> echo $gzip\_contents;
> echo pack("V", $gzip\_crc);
> echo pack("V", $gzip\_size);
> }
> exit;
}

function get\_calendar\_js($id, $value)
{
> $cleanname = preg\_replace("/[^-\._a-zA-Z0-9]/", "", $id);
?>_

&lt;script language="JavaScript" type="text/javascript"&gt;


var root\_path = '<?php echo ROOT\_PATH; ?>';
<!--
function calendarSetDate_<?php echo $cleanname; ?>(day, month, year)
{
> var split = document.getElementById('<?php echo $id; ?>').value.split(' ');
> var currTime = split[1](1.md) ? split[1](1.md) : '';_

> if (day < 10) {
> > day = '0' + day;

> }

> if (month < 10) {
> > month = '0' + month;

> }

> var value = year + '-' + month + '-' + day;

> if (currTime) {
> > value += ' ' + currTime;

> }

> document.getElementById('<?php echo $id; ?>').value = value;
}

var calendar_<?php echo $cleanname; ?> = new calendar('calendar_<?php echo $cleanname; ?>', 'calendarSetDate_<?php echo $cleanname; ?>');
<?php
if ($value) {
?>
calendar_<?php echo $cleanname; ?>.setCurrentMonth(<?php echo intval(date('m', strtotime($value))); ?>);
calendar_<?php echo $cleanname; ?>.setCurrentYear(<?php echo date('Y', strtotime($value)); ?>);
<?php
}
?>
calendar_<?php echo $cleanname; ?>.writeHTML();
//-->


&lt;/script&gt;


<?php

}

function get\_row\_bg() {
> global $bgcounter;
> return ($bgcounter++ % 2 == 0) ? "tablerow" : "tablerow2";
}

function show\_table\_header($title, $colspan = 2, $anchor = "") {
> global $bgcounter;
> echo "<table cellpadding='\"3\"' cellspacing='\"1\"' border='\"0\"' width='\"100%\"'>\n";<br>
<blockquote>echo "<tr colspan='\"$colspan\"><a'>";<br>
echo $title;<br>
echo "<br>
<br>
Unknown end tag for </span><br>
<br>
<br>
<br>
Unknown end tag for </b><br>
<br>
<br>
<br>
Unknown end tag for </a><br>
<br>
\n<br>
<br>
Unknown end tag for </td><br>
<br>
\n</tr>\n";<br>
$bgcounter = 0;<br>
}</blockquote>

function show_table_separator($title, $colspan = 2, $anchor = "") {<br>
<blockquote>global $bgcounter;<br>
echo "<tr colspan='\"$colspan\"><a'>\n";<br>
$bgcounter = 0;<br>
}</blockquote>

function show_table_footer() {<br>
<blockquote>echo "<br>
<br>
Unknown end tag for </table><br>
<br>
\n<br>
<br>
Unknown end tag for </td><br>
<br>
\n</tr>\n</table><br />\n";<br>
}</blockquote>

function show\_form\_header($phpscript, $action = "", $name = "formular", $uploadform = 0) {
> global $site\_sess;

> if ($uploadform) {
> > $upload = " ENCTYPE=\"multipart/form-data\"";

> }
> else {
> > $upload = "";

> }
> echo "

&lt;form action=\"".$site\_sess-&gt;url(safe\_htmlspecialchars(strip\_tags($phpscript)))."\"".$upload." name=\"".$name."\" method=\"post\"&gt;

\n";
> if ($action != "") {
> > echo "

&lt;input type=\"hidden\" name=\"action\" value=\"".$action."\"&gt;

\n";

> }
}

function show\_form\_footer($submitname = "Submit", $resetname = "Reset", $colspan = 2, $goback = "", $javascript = "") {
> echo "<tr colspan='\"".$colspan."\"' align='\"center\"'>\n&nbsp;";<br>
<blockquote>if ($submitname != "") {<br>
<blockquote>echo "<input type=\"submit\" value=\"   ".$submitname."   \" class=\"button\"";<br>
if ($javascript != "") {<br>
<blockquote>echo " ".$javascript;<br>
</blockquote>}<br>
echo ">\n";<br>
</blockquote>}<br>
if ($resetname != "") {<br>
<blockquote>echo "<input type=\"reset\" value=\"   ".$resetname."   \" class=\"button\">\n";<br>
</blockquote>}<br>
if ($goback != "") {<br>
echo "<input type=\"submit\" name=\"goback\" value=\"   ".$goback."   \" class=\"button\">\n";<br>
}<br>
echo "&nbsp;\n<br>
<br>
Unknown end tag for </td><br>
<br>
\n</tr>\n<br>
<br>
Unknown end tag for </table><br>
<br>
\n<br>
<br>
Unknown end tag for </td><br>
<br>
\n<br>
<br>
Unknown end tag for </tr><br>
<br>
\n<br>
<br>
Unknown end tag for </table><br>
<br>
\n<br>
<br>
Unknown end tag for </form><br>
<br>
\n";<br>
}</blockquote>

function show\_custom\_row($title, $value) {
> echo "<tr valign='\"top\">\n<td><p'>\n";<br>
}</li></ul>

function show_num_select_row($title, $option, $desc = "") {<br>
<blockquote>global $site\_sess, $PHP\_SELF, $action, $$option;
> echo "<tr>\n";<br>
echo "<td align='\"right\"><p'>".$desc;<br>
$url = $PHP_SELF;<br>
$url .= preg_match("/\?/", $url) ? "&amp;" : "?";<br>
$url .= "action=".$action;<br>
$url = $site_sess->url($url);<br>
echo "<select name=\"num\" onchange=\"window.location=('".$url."&";<br>
echo $option."='+this.options[this.selectedIndex].value)\">\n";<br>
for ($i = 1; $i < 11; $i++) {<br>
<blockquote>echo "<option value=\"$i\"";<br>
if ($i == ${$option}) {<br>
<blockquote>echo " selected";<br>
</blockquote>}<br>
echo ">".$i."<br>
<br>
Unknown end tag for </option><br>
<br>
\n";<br>
</blockquote>}<br>
echo "<br>
<br>
Unknown end tag for </select><br>
<br>
<br>
<br>
Unknown end tag for </p><br>
<br>
</td>\n</tr>\n";
}

function show\_upload\_row($title, $name, $extra = "", $value = "") {
> global $error, $HTTP\_POST\_VARS, $textinput\_size;
> if (isset($error[$name]) || isset($error['remote_'.$name])) {
> > $title = sprintf("_<span>%s **</span>", $title);

> }
> if (isset($HTTP\_POST\_VARS['remote**'.$name])/** && $value == ""_/) {
> > $value = stripslashes($HTTP\_POST\_VARS['remote_'.$name]);

> }**

> echo "<tr valign='top'>\n<td><p>\n";<br>
echo "<td><p>";<br>
echo "<b>Upload:</b><br>

<input type=\"file\" name=\"".$name."\"><br>

";<br>
echo "<b>URL:</b><br>

<input type=\"text\" name=\"remote_".$name."\" value=\"".$value."\" size=\"".$textinput_size."\">

";<br>
echo $extra."<br>
<br>
Unknown end tag for </p><br>
<br>
<br>
<br>
Unknown end tag for </td><br>
<br>
\n<br>
<br>
Unknown end tag for </tr><br>
<br>
\n";<br>
}</blockquote>

function show_image_row($title, $src, $border = 0, $delete_box = "", $height = 0, $width = 0) {<br>
<blockquote>global $HTTP_POST_VARS, $lang;<br>
$dimension = "";<br>
if ($height) {<br>
<blockquote>$dimension .= " height=\"".$height."\"";<br>
</blockquote>}<br>
if ($width) {<br>
<blockquote>$dimension .= " width=\"".$width."\"";<br>
</blockquote>}<br>
echo "<tr valign='top'>\n<td><p>\n";<br>
echo "<td><img src='\"".$src."\"".$dimension."' alt='\"\"' border='\"".$border."\"'>";<br>
if ($delete_box != "") {<br>
<blockquote>$checked = '';<br>
if (isset($HTTP_POST_VARS[$delete_box]) && $HTTP_POST_VARS[$delete_box] == 1) {<br>
<blockquote>$checked = ' checked="checked"';<br>
</blockquote>}<br>
echo "<br>
<br>
<input type=\"checkbox\" name=\"".$delete_box."\" value=\"1\"".$checked."><br>
<br>
 ".$lang['delete'];<br>
</blockquote>}<br>
echo "<br>
<br>
Unknown end tag for </td><br>
<br>
\n<br>
<br>
Unknown end tag for </tr><br>
<br>
\n";<br>
}</blockquote>

function show_description_row($text, $colspan = 2) {<br>
<blockquote>echo "<tr colspan='\"".$colspan."\">".$text."</td>\n</tr'>\n";<br>
}</blockquote>

function show_radio_row($title, $name, $value = 1) {<br>
<blockquote>global $HTTP_POST_VARS, $lang;<br>
if (isset($HTTP_POST_VARS[$name])) {<br>
<blockquote>$value = $HTTP_POST_VARS[$name];<br>
</blockquote>}<br>
echo "<tr>\n";<br>
echo "<td><p>";<br>
echo "<input type=\"radio\" name=\"$name\" value=\"1\"";<br>
if ($value == 1) {<br>
<blockquote>echo " checked=\"checked\"";<br>
</blockquote>}<br>
echo "> ".$lang['yes']."&nbsp;&nbsp;&nbsp;\n";<br>
echo "<input type=\"radio\" name=\"".$name."\" value=\"0\"";<br>
if ($value != 1) {<br>
<blockquote>echo " checked=\"checked\"";<br>
</blockquote>}<br>
echo "> ".$lang['no']." ";<br>
echo "</p></td>\n</tr>";<br>
}</blockquote>

function show_input_row($title, $name, $value = "", $size = "") {<br>
<blockquote>global $error, $HTTP_POST_VARS, $textinput_size;<br>
$size = (empty($size)) ? $textinput_size : $size;<br>
if (isset($error[$name])) {<br>
<blockquote>$title = sprintf("<span>%s <b></span>", $title);<br>
</blockquote>}<br>
if (isset($HTTP_POST_VARS[$name])/</b> && $value == ""<b>/) {<br>
<blockquote>$value = stripslashes($HTTP_POST_VARS[$name]);<br>
</blockquote>}<br>
echo "</b><tr><input type=\"text\" size=\"".$size."\" name=\"".$name."\" value=\"".format_text($value, 2)."\"><br>
<br>
Unknown end tag for </p><br>
<br>
<br>
<br>
Unknown end tag for </td><br>
<br>
\n</tr>\n";<br>
}</blockquote>

function show_date_input_row($title, $name, $value = "", $size = "") {<br>
<blockquote>global $error, $HTTP_POST_VARS, $textinput_size;<br>
$size = (empty($size)) ? $textinput_size : $size;<br>
if (isset($error[$name])) {<br>
<blockquote>$title = sprintf("<span>%s <b></span>", $title);<br>
</blockquote>}<br>
if (isset($HTTP_POST_VARS[$name])/</b> && $value == ""<b>/) {<br>
<blockquote>$value = stripslashes($HTTP_POST_VARS[$name]);<br>
</blockquote>}</blockquote></b>

<blockquote>echo "<tr><input type=\"text\" size=\"".$size."\" name=\"".$name."\" id=\"".$name."\" value=\"".format_text($value, 2)."\"> ";<br>
echo get_calendar_js($name, $value);<br>
echo "<br>
<br>
Unknown end tag for </td><br>
<br>
\n</tr>\n";<br>
}</blockquote>

function show_textarea_row($title, $name, $value = "", $cols = "", $rows = 10) {<br>
<blockquote>global $error, $HTTP_POST_VARS, $textarea_size;<br>
$cols = (empty($cols)) ? $textarea_size : $cols;<br>
if (isset($error[$name])) {<br>
<blockquote>$title = sprintf("<span>%s <b></span>", $title);<br>
</blockquote>}<br>
if (isset($HTTP_POST_VARS[$name])/</b> && $value == ""<b>/) {<br>
<blockquote>$value = stripslashes($HTTP_POST_VARS[$name]);<br>
</blockquote>}<br>
echo "</b><tr valign='\"top\">\n<td><p'>".format_text($value, 2)."<br>
<br>
Unknown end tag for </textarea><br>
<br>
<br>
<br>
Unknown end tag for </p><br>
<br>
<br>
<br>
Unknown end tag for </td><br>
<br>
\n</tr>\n";<br>
}</blockquote>

function show_user_select_row($title, $user_id, $i = 0) {<br>
<blockquote>global $error, $lang, $HTTP_POST_VARS, $site_db, $user_table_fields, $user_select_row_cache;</blockquote>

<blockquote>if (empty($user_select_row_cache)) {<br>
<blockquote>$sql = "SELECT ".get_user_table_field("", "user_id").get_user_table_field(", ", "user_name")."<br>
<blockquote>FROM ".USERS_TABLE."<br>
WHERE ".get_user_table_field("", "user_id")." <> ".GUEST."<br>
ORDER BY ".get_user_table_field("", "user_name")." ASC";<br>
</blockquote>$result = $site_db->query($sql);<br>
$user_select_row_cache = array();<br>
while ($row = $site_db->fetch_array($result)) {<br>
<blockquote>$user_select_row_cache[$row[$user_table_fields['user_id']]] = $row[$user_table_fields['user_name']];<br>
</blockquote>}<br>
</blockquote>}</blockquote>

<blockquote>if (isset($error['user_id<i>'.$i]) || isset($error['user_id'])) {<br>
<blockquote>$title = sprintf("</i><span>%s <i></span>", $title);<br>
</blockquote>}<br>
if (isset($HTTP_POST_VARS['user_id<i>'.$i])) {<br>
<blockquote>$user_id = $HTTP_POST_VARS['user_id</i>'.$i];<br>
</blockquote>}<br>
elseif (isset($HTTP_POST_VARS['user_id'])) {<br>
<blockquote>$user_id = $HTTP_POST_VARS['user_id'];<br>
</blockquote>}<br>
$i = ($i) ? "<b>".$i : "";<br>
echo "</b><tr>\n";<br>
echo "</i><td>\n";<br>
echo "<br>
<br>
<select name=\"user_id".$i."\" class=\"categoryselect\"><br>
<br>
\n";<br>
echo "<br>
<br>
<option value=\"".GUEST."\">".$lang['userlevel_guest']."</option><br>
<br>
\n";<br>
echo "<br>
<br>
<option value=\"".GUEST."\">-------------------------------</option><br>
<br>
\n";<br>
foreach ($user_select_row_cache as $key => $val) {<br>
<blockquote>echo "<option value=\"".$key."\"";<br>
if ($key == $user_id) {<br>
<blockquote>echo " selected=\"selected\"";<br>
</blockquote>}<br>
echo ">".format_text($val, 2)."<br>
<br>
</option><br>
<br>
\n";<br>
</blockquote>}<br>
echo "<br>
<br>
Unknown end tag for </select><br>
<br>
\n";<br>
echo "<br>
<br>
Unknown end tag for </td><br>
<br>
\n<br>
<br>
Unknown end tag for </tr><br>
<br>
\n";<br>
}</blockquote>

function show_cat_select_row($title, $cat_id, $admin = 0, $i = 0) {<br>
<blockquote>global $error, $HTTP_POST_VARS;<br>
if (isset($error['cat_id<i>'.$i])</i><table><thead><th> isset($error['cat_id']) || isset($error['cat_parent_id'])) {</th></thead><tbody>
<blockquote>$title = sprintf("<span>%s <b></span>", $title);<br>
</blockquote>}<br>
if (isset($HTTP_POST_VARS['cat_parent_id'])) {<br>
<blockquote>$cat_id = $HTTP_POST_VARS['cat_parent_id'];<br>
</blockquote>}<br>
elseif (isset($HTTP_POST_VARS['cat_id<i>'.$i])) {<br>
<blockquote>$cat_id = $HTTP_POST_VARS['cat_id</i>'.$i];<br>
</blockquote>}<br>
elseif (isset($HTTP_POST_VARS['cat_id'])) {<br>
<blockquote>$cat_id = $HTTP_POST_VARS['cat_id'];<br>
</blockquote>}<br>
echo "</b><tr>".get_category_dropdown($cat_id, 0, $admin, $i)."<br>
<br>
Unknown end tag for </td><br>
<br>
\n</tr>\n";<br>
}</blockquote></tbody></table>

function show_userlevel_select_row($title, $name = "user_level", $userlevel = "") {<br>
<blockquote>global $lang, $error, $HTTP_POST_VARS;<br>
if (isset($error[$name])) {<br>
<blockquote>$title = sprintf("<span>%s <b></span>", $title);<br>
</blockquote>}<br>
if (isset($HTTP_POST_VARS[$name])/</b> && $userlevel == ""<b>/) {<br>
<blockquote>$userlevel = stripslashes($HTTP_POST_VARS[$name]);<br>
</blockquote>}<br>
echo "</b><tr>\n";<br>
echo "<br>
<br>
<select name=".$name."><br>
<br>
\n";<br>
echo "<option value=\"".GUEST."\"";<br>
if ($userlevel == GUEST || $userlevel == "") {<br>
<blockquote>echo " selected=\"selected\"";<br>
</blockquote>}<br>
echo ">--<br>
<br>
Unknown end tag for </option><br>
<br>
\n";<br>
echo "<option value=\"".ADMIN."\"";<br>
if ($userlevel == ADMIN && $userlevel != "") {<br>
<blockquote>echo " selected=\"selected\"";<br>
</blockquote>}<br>
echo ">".$lang['userlevel_admin']."<br>
<br>
Unknown end tag for </option><br>
<br>
\n";<br>
echo "<option value=\"".USER."\"";<br>
if ($userlevel == USER && $userlevel != "") {<br>
<blockquote>echo " selected=\"selected\"";<br>
</blockquote>}<br>
echo ">".$lang['userlevel_registered']."<br>
<br>
Unknown end tag for </option><br>
<br>
\n";<br>
echo "<option value=\"".USER_AWAITING."\"";<br>
if ($userlevel == USER_AWAITING && $userlevel != "") {<br>
<blockquote>echo " selected=\"selected\"";<br>
</blockquote>}<br>
echo ">".$lang['userlevel_registered_awaiting']."<br>
<br>
Unknown end tag for </option><br>
<br>
\n";<br>
echo "<br>
<br>
</select><br>
<br>
\n<br>
<br>
Unknown end tag for </td><br>
<br>
\n</tr>\n";<br>
}</blockquote>

function show_hidden_input($name, $value = "", $htmlise = 1) {<br>
<blockquote>if ($htmlise) {<br>
<blockquote>$value = format_text($value, 2);<br>
</blockquote>}<br>
echo "<br>
<br>
<input type=\"hidden\" name=\"$name\" value=\"".$value."\"><br>
<br>
\n";<br>
}</blockquote>

function show_text_link($text, $url, $newwin = 0) {<br>
<blockquote>global $site_sess;<br>
$target = ($newwin) ? " target=\"<i>blank\"" : "";<br>
echo "</i><a href='\"".$site_sess->url(safe_htmlspecialchars(strip_tags($url)))."\"".$target.">[".$text."]</a'>&nbsp;&nbsp;";<br>
}</blockquote>

function get_navrow_bg() {<br>
<blockquote>global $navbgcounter;<br>
return ($navbgcounter++ % 2 == 0) ? "#E5E5E5" : "#F5F5F5";<br>
}</blockquote>

function show_nav_option($title, $url, $extra = "")  {<br>
<blockquote>global $site_sess;<br>
$bgcolor = get_navrow_bg();<br>
echo "<tr><td valign='top'>\n";<br>
echo "<table cellpadding='\"3\"' border='\"0\"' cellspacing='\"0\"><tr><td'>\n";<br>
echo "<a href='\"".$site_sess->url(safe_htmlspecialchars(strip_tags($url)))."\"'> $extra\n";<br>
echo "<br>
<br>
Unknown end tag for </td><br>
<br>
<br>
<br>
Unknown end tag for </tr><br>
<br>
<br>
<br>
Unknown end tag for </table><br>
<br>
\n";<br>
echo "<br>
<br>
Unknown end tag for </td><br>
<br>
<br>
<br>
Unknown end tag for </tr><br>
<br>
\n";<br>
echo "<tr><td>\n";<br>
}</blockquote>

function show_nav_header($title)  {<br>
<blockquote>global $navbgcounter;<br>
echo "<tr><td>\n";<br>
echo "<table cellpadding='\"3\"' border='\"0\"' cellspacing='\"0\"><tr><td'>\n";<br>
echo $title;<br>
echo "<br>
<br>
Unknown end tag for </td><br>
<br>
<br>
<br>
Unknown end tag for </tr><br>
<br>
</table>\n";<br>
echo "</td></tr>\n";<br>
echo "<tr><td>\n";<br>
$navbgcounter = 0;<br>
}</blockquote>

function check_admin_date($date) {<br>
<blockquote>return (preg_match('/^([0-9]{4})-([0-9]{2})-([0-9]{2})+(( )+[0-9]{2}:[0-9]{2}:([0-9]{2}))?$/', stripslashes(trim($date)))) ? 1 : 0;<br>
}</blockquote>

function get_dir_size($dir) {<br>
<blockquote>$size = 0;<br>
$dir = (!preg_match("#/$#", $dir)) ? $dir."/" : $dir;<br>
$handle = @opendir($dir);<br>
while ($file = @readdir($handle)) {<br>
<blockquote>if (preg_match("/^\.{1,2}$/",$file)) {<br>
<blockquote>continue;<br>
</blockquote>}<br>
$size += (is_dir($dir.$file)) ? get_dir_size($dir.$file."/") : filesize($dir.$file);<br>
</blockquote>}<br>
@closedir($handle);<br>
return $size;<br>
}</blockquote>

function show_additional_fields($type = "image", $image_row = array(), $table = IMAGES_TEMP_TABLE, $i = 0) {<br>
<blockquote>global $site_db, $lang;</blockquote>

<blockquote>$field_type_array = "additional<i>".$type."</i>fields";<br>
global ${$field_type_array};</blockquote>

<blockquote>if (!empty(${$field_type_array})) {<br>
<blockquote>$table_fields = $site_db->get_table_fields($table);<br>
foreach (${$field_type_array} as $key => $val) {<br>
<blockquote>if (!isset($table_fields[$key])) {<br>
<blockquote>continue;<br>
</blockquote>}<br>
$field_name = ($i) ? $key."