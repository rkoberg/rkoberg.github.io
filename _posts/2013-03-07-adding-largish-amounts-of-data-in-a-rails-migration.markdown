---
layout: post
status: publish
published: true
title: Adding largish amounts of data in a rails migration
author:
  display_name: Rob Koberg
  login: rkoberg
  email: rob.koberg@gmail.com
  url: ''
author_login: rkoberg
author_email: rob.koberg@gmail.com
wordpress_id: 45
wordpress_url: http://www.koberg.com/?p=45
date: '2013-03-07 21:13:14 -0800'
date_gmt: '2013-03-07 21:13:14 -0800'
tags:
- rails migrate database postgresql
comments: []
---
<p>There was a little trick to it. First, here is the migrate script:</p>
<pre>class ImportMdrDataToSchoolsImport &lt; ActiveRecord::Migration
  def up
    ActiveRecord::Base.connection.execute(IO.read("./db/schools_import.sql"))
  end

  def down
  end
end</pre>
<p>The schools_import.sql file is about 75MB.</p>
<p>The trick was to use pg_dump with the <code>--inserts</code> flag as using a standard dump's stdin would not work.</p>
<pre>pg_dump --inserts -t schools_import schools &gt; schools_import.sql</pre>
<p>I also edited the dump to remove everything but the insert statements and the create indexes. Then I added a statement for truncating the table, then drop the indexes, then the existing insert statements from the dump followed by the create indexes. For example:</p>
<pre>TRUNCATE TABLE schools_import;

DROP INDEX "index_schools_import_on_MCITY";
DROP INDEX "index_schools_import_on_MSTATE";

INSERT INTO schools_import VALUES (...)
...all inserts...

CREATE INDEX "index_schools_import_on_MCITY" ON schools_import USING btree ("MCITY");
CREATE INDEX "index_schools_import_on_MSTATE" ON schools_import USING btree ("MSTATE");</pre>
<p>It took under 8 minutes to run and put about 260k records.</p>
